# 10 — Troubleshooting (Problemas Comuns)

> FAQ organizado por sintoma. Acha o que tá acontecendo, segue os passos.

---

## Como usar esse documento

1. Identifica o sintoma (a coisa que tá errada)
2. Acha a seção correspondente abaixo
3. Segue os passos de **diagnóstico** primeiro (não tenta corrigir antes de entender)
4. Aplica a **solução** quando souber a causa
5. **Valida** que voltou ao normal

> 💡 Se nada aqui resolver: pega os logs (`pm2 logs <processo> --lines 50 --nostream`) e pede ajuda ao Claude descrevendo o sintoma + logs.

---

## 🔴 "O Hub não responde"

### Sintoma
URL drosagencia.com.br/hub não carrega ou retorna 502/504.

### Diagnóstico
```bash
ssh root@vps-...
pm2 status
```

| Status | Significado |
|---|---|
| `online` | PM2 acha que tá rodando — investigar logs |
| `errored` | Processo quebrou no boot |
| `stopped` | Alguém parou e não subiu de volta |

```bash
pm2 logs dros-hub --lines 50 --nostream --err
```

### Soluções

**Se `stopped`:**
```bash
pm2 start dros-hub
```

**Se `errored` ou erro nos logs:**
- `SyntaxError` → código tem bug, fazer rollback (`git revert HEAD; pm2 stop dros-hub && pm2 start dros-hub`)
- `EADDRINUSE` (porta em uso) → outro processo na 3003: `lsof -i :3003` → mata
- `Cannot find module` → faltou `npm install`: `cd /opt/platform/agency-hub && npm install`
- `SqliteError: no such column` → migration faltou: ver [Receita 42](12-RECEITAS-PASSO-A-PASSO.md#receita-42)

---

## 🟠 "Token Meta expirou" (Session expired)

### Sintoma
Hub: Performance mostra "Sem dados" ou erro.
Logs:
```
Error validating access token: Session has expired on Monday, XX-XXX-XX 11:49:38 PDT
```

### Causa
Token Meta dura **60 dias**. Vence em torno da última renovação.

### Solução rápida (UI)
1. Hub → Configurações → Tokens
2. Meta Access Token → Trocar → cola novo token → Salvar

### Como gerar token novo
1. https://developers.facebook.com/tools/explorer/
2. Selecionar app "Cloude_app_DROS"
3. Obter Token de Acesso do Usuário, scopes: `ads_read`, `ads_management`, `business_management`, `pages_show_list`, `pages_read_engagement`, `instagram_basic`
4. Token retornado é curto. Estender pra 60 dias:
```bash
curl "https://graph.facebook.com/v21.0/oauth/access_token?grant_type=fb_exchange_token&client_id=944656904876695&client_secret=<SECRET>&fb_exchange_token=<TOKEN_CURTO>"
```
5. Cola o `access_token` longo na UI

### Alternativa permanente
System User Token no Business Manager — **não vence**. Veja [Receita 8](12-RECEITAS-PASSO-A-PASSO.md#receita-8).

---

## 🟠 "Token Meta inválido" (Cannot parse access token)

### Sintoma
Diferente do anterior — token novo, mas Meta diz que não consegue parsear.

### Causa típica
Token gravado com caractere extra. Já tivemos:
- `==` duplo no `.env` (typo no nano)
- Espaço/newline no início ou fim
- Aspas duplas no valor

### Diagnóstico
```bash
TOKEN=$(sqlite3 /opt/platform/agency-hub/server/data/hub.db "SELECT value FROM app_settings WHERE key='META_ACCESS_TOKEN';")
echo "Primeiros 20: ${TOKEN:0:20}"
echo "Length: ${#TOKEN}"
```

Token bom começa com `EAA...`. Se começar com `=EAA...`, tem o bug.

### Solução
```bash
# Remove o = inicial
sqlite3 /opt/platform/agency-hub/server/data/hub.db \
  "UPDATE app_settings SET value = substr(value, 2) WHERE key='META_ACCESS_TOKEN' AND substr(value, 1, 1) = '=';"

# Limpa .env também se afetado
sed -i 's/^META_ACCESS_TOKEN==/META_ACCESS_TOKEN=/' /opt/platform/agency-hub/.env

# Testa
TOKEN=$(sqlite3 /opt/platform/agency-hub/server/data/hub.db "SELECT value FROM app_settings WHERE key='META_ACCESS_TOKEN';")
curl -s "https://graph.facebook.com/v21.0/me?access_token=$TOKEN" | head -c 200
```

Se retornar `{"name": "...", "id": "..."}` — fix funcionou.

---

## 🟡 "/core mostra sidebar vazio"

### Sintoma
Entra em `drosagencia.com.br/core/`, sidebar não lista contas. Só "Buscar conta...".

### Diagnóstico 1: cache do navegador
```
DevTools (F12) → Network → Disable cache → Ctrl+Shift+R
```

Se voltar — era cache.

### Diagnóstico 2: API funcionando?
```bash
# VPS
ssh root@vps-...

# Login e pega token
TOKEN_CORE=$(curl -s -X POST http://localhost:3004/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@drosagencia.com.br","password":"dros2026"}' | grep -oP '"token":"[^"]+' | cut -d'"' -f4)

# Testa accounts
curl -s -H "Authorization: Bearer $TOKEN_CORE" "http://localhost:3004/api/meta/accounts" | head -c 500
```

Se retornar `{"accounts":[...]}` com contas — backend OK, problema é frontend (cache).

Se retornar `{"error": "..."}` — backend tem problema (token, Meta API, etc).

### Diagnóstico 3: sync do Hub funcionando?
```bash
pm2 logs dros-core --lines 30 --nostream | grep -i "hub sync"
```

Esperado:
```
[Hub sync] tokens atualizados X chaves
[Hub sync] clientes recebidos: Y
```

Se não aparece, sync tá falhando — verificar HUB_URL e CORE_EMBED_SECRET.

### Solução geral
- Hard refresh navegador
- Service Worker → Unregister (DevTools → Application)
- Clear site data (DevTools → Application → Clear storage)

---

## 🟡 "/core retorna PathError" (Express 5 wildcard)

### Sintoma
Logs do /core:
```
PathError [TypeError]: Missing parameter name at index 7: /core/*; visit https://git.new/pathToRegexpError
```

### Causa
Express 5 mudou a sintaxe de wildcard. `/core/*` não funciona mais.

### Solução
Em `client-dashboard/server/index.js`, mudar:
```js
app.get('/core/*', (req, res) => { ... })          // ❌ velho
app.get('/core/{*path}', (req, res) => { ... })    // ✅ novo
```

Já está corrigido no código atual (commit `cb34a19`).

---

## 🔴 "Disco cheio"

### Sintoma
- Postgres do Evolution crasha
- Não consegue salvar arquivos
- PM2 logs param
- `df -h` mostra > 85% de uso

### Diagnóstico
```bash
df -h
du -sh /* 2>/dev/null | sort -rh | head -10
```

Geralmente o vilão é:
- `/var/lib/docker` — logs e volumes do Evolution
- `/root/.pm2/logs` — logs antigos do PM2
- `/opt/platform/agency-hub/server/data/` — backups antigos
- `/var/log` — logs do sistema

### Soluções

**Logs do PM2:**
```bash
pm2 flush
```

**Backups antigos do DB:**
```bash
find /opt/platform -name "*.backup*" -mtime +30 -size +10M -delete
```

**Docker (Evolution):**
```bash
docker system prune -af --volumes  # cuidado, remove volumes não usados
```

**Logs do sistema:**
```bash
journalctl --vacuum-time=7d  # mantém só últimos 7 dias
```

---

## 🟠 "CRM não recebe leads"

### Sintoma
Form do site é submetido, mas lead não aparece no CRM.

### Diagnóstico 1: Apps Script trigger rodando?
1. Abre a planilha do cliente (ex: "ENTRADA DE LEADS TELHABRAS")
2. Extensões → Apps Script
3. Menu lateral: Triggers
4. Confirmar trigger `sincronizarLeadsNovos` ativo, a cada 5min

Se não tem, rodar `instalarTrigger()` (botão Run no editor).

### Diagnóstico 2: Lead chegou na planilha?
Abre a planilha → aba "FORM SITE" → confirmar que a linha foi adicionada.

Se não → problema no form (frontend) ou no `/exec` do Apps Script.

### Diagnóstico 3: Apps Script consegue chamar o CRM?
No Apps Script: View → Executions → vê última execução do `sincronizarLeadsNovos`.

Se tem erro 401/403/404 → URL errada ou autenticação. Verificar `CRM_WEBHOOK` no Apps Script.

Se tem 500 → CRM caiu ou bug. Logs:
```bash
pm2 logs dros-crm --lines 30 --nostream --err
```

### Diagnóstico 4: Evolution rodando?
```bash
docker ps | grep evolution
```

Se algum container down: `docker start evolution-api evolution-redis evolution-postgres`.

---

## 🟠 "Performance mostra 'Sem dados'"

### Sintoma
Hub → Performance → cliente X aparece com cards zerados ou "Sem dados".

### Diagnóstico
1. **Token Meta vencido?** Configurações → Tokens → verifica status
2. **IDs do cliente preenchidos?** Configurações → Contas → confirma badges Meta/IG/GAds/GA4
3. **API funcionando?** Logs:
```bash
pm2 logs dros-hub --lines 30 --nostream | grep -i "performance"
```

### Solução típica
- Renovar token (ver acima)
- Confirmar IDs no cadastro do cliente
- Se IDs estão certos mas não retorna dados: verificar se a conta Meta realmente tem gasto no período (cliente novo pode estar zerado)

---

## 🔴 "DB corrompido" (raro)

### Sintoma
Erro tipo: `SqliteError: database disk image is malformed`.

### Causa
Crash durante write no DB. Acontece se VPS reinicia abrupto.

### Solução
**1. Parar Hub:**
```bash
pm2 stop dros-hub
```

**2. Tenta recuperar:**
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db ".recover" > /tmp/recovered.sql
sqlite3 /opt/platform/agency-hub/server/data/hub_recovered.db < /tmp/recovered.sql
```

Se gerou um DB OK: copia por cima do original.
```bash
cp /opt/platform/agency-hub/server/data/hub.db /opt/platform/agency-hub/server/data/hub.db.corrupted
cp /opt/platform/agency-hub/server/data/hub_recovered.db /opt/platform/agency-hub/server/data/hub.db
```

**3. Se não recupera: restaurar último backup**
```bash
cp /opt/platform/agency-hub/server/data/hub.db.backup-XXX /opt/platform/agency-hub/server/data/hub.db
```

**4. Sobe Hub:**
```bash
pm2 start dros-hub
```

> ⚠️ Restaurar backup **descarta dados desde o backup**. Aceitável quando alternativa é DB perdido.

---

## 🟡 "PM2 não pegou variável de ambiente nova"

### Sintoma
Editou `.env`, fez `pm2 restart dros-hub`, mas processo ainda usa valor velho.

### Causa
`pm2 restart` às vezes não recarrega env corretamente.

### Solução
**Sempre prefira stop+start:**
```bash
pm2 stop dros-hub && pm2 start dros-hub
```

**Ou força update env:**
```bash
pm2 restart dros-hub --update-env
```

---

## 🟠 "Schema parcial de tabela" (raro)

### Sintoma
Erro tipo: `SqliteError: table X has no column named Y`.

### Causa
Migration parcial ou interrompida. Tabela existe mas faltam colunas.

### Diagnóstico
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db "PRAGMA table_info(X);"
```

Compara com schema esperado em `db.js`.

### Solução
Já temos auto-detection no `db.js` que detecta tabelas com schema incompleto e recria (drop + create). Aconteceu com `task_templates` em maio/2026.

Se não foi auto-resolvido:
```bash
# Backup primeiro
cp /opt/platform/agency-hub/server/data/hub.db /opt/platform/agency-hub/server/data/hub.db.backup-$(date +%Y%m%d-%H%M)

# Drop manual e deixa migration recriar
sqlite3 /opt/platform/agency-hub/server/data/hub.db "DROP TABLE IF EXISTS X;"

# Restart pra rodar migration
pm2 stop dros-hub && pm2 start dros-hub
```

---

## 🟢 "Notificações pararam de aparecer em tempo real"

### Sintoma
Sino do Hub não atualiza contador sem reload manual.

### Causa
SSE (Server-Sent Events) desconectou. Acontece após:
- Internet do usuário cai e volta
- Servidor restartou
- Proxy reverso (Cloudflare/Apache) reseta conexão longa

### Solução
1. **Cliente:** Ctrl+Shift+R (reconecta SSE)
2. **Se persistir:** `pm2 stop dros-hub && pm2 start dros-hub`
3. **Diagnóstico SSE:**
```bash
pm2 logs dros-hub --lines 30 --nostream | grep -i "sse\|notification"
```

---

## 🟡 "Build do frontend falhou"

### Sintoma
`npm run build` retorna erro.

### Causas comuns

**TypeScript error:**
```
src/X.tsx(10,5): error TS2322: Type 'string' is not assignable to type 'number'
```
→ tipo errado. Mostra pro Claude → ele conserta.

**Module not found:**
```
Cannot find module '@/components/X'
```
→ dependência não instalada ou path errado. `npm install` resolve dependência. Path errado, Claude corrige.

**Out of memory:**
```
JavaScript heap out of memory
```
→ build estourou RAM. Tenta:
```bash
NODE_OPTIONS=--max-old-space-size=4096 npm run build
```

---

## 🟢 "Subi código mas o navegador continua mostrando o antigo"

Cache. Sempre cache.

1. Ctrl+Shift+R no navegador
2. Se persiste: F12 → Network → Disable cache → recarrega
3. Se persiste: Service Worker → Unregister (DevTools → Application)
4. Se persiste: Clear site data

Confirma que `npm run build` rodou local e `dist/` foi commitado.

---

## Quando nada resolve

1. **Para de tentar consertar.** Faz rollback do último commit:
   ```bash
   git revert HEAD
   git push <remote> master
   # VPS: git pull + pm2 stop+start
   ```

2. **Coleta evidências:**
   - Screenshot do erro
   - Output do `pm2 status`
   - Últimos logs (`pm2 logs --lines 100 --nostream`)
   - Comando que disparou o problema

3. **Pede ajuda** com tudo isso ao Claude (ou ao João Luiz se for fora do horário).

4. **Documenta** o problema + solução em [HISTORICO](HISTORICO.md) depois.

---

## Próximo passo

[11 — Glossário](11-GLOSSARIO.md) — todos os termos técnicos explicados.
