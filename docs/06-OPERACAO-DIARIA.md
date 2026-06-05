# 06 — Operação Diária

> Comandos que você vai precisar no dia a dia. Copia, cola, funciona.

---

## Acessar a VPS

```bash
ssh root@vps-5269157.3store.com.br
```

Senha ou chave: configurada uma vez. Veja [Receita 4](12-RECEITAS-PASSO-A-PASSO.md#receita-4) se nunca configurou.

Pra sair: `exit` ou `Ctrl+D`.

---

## Comandos PM2 (gerenciador de processos)

### Ver tudo rodando
```bash
pm2 status
```

Saída típica:
```
┌─────┬──────────────────┬─────────┬─────────┬─────────┐
│ id  │ name             │ status  │ cpu     │ memory  │
├─────┼──────────────────┼─────────┼─────────┼─────────┤
│ 0   │ dros-hub         │ online  │ 0%      │ 70mb    │
│ 10  │ dros-core        │ online  │ 0%      │ 33mb    │
│ 12  │ dros-crm         │ online  │ 0%      │ 92mb    │
│ 15  │ dros-oxi-pedidos │ online  │ 0%      │ 17mb    │
│ 6   │ gestao-clin      │ online  │ 0%      │ 26mb    │
└─────┴──────────────────┴─────────┴─────────┴─────────┘
```

Coluna `status` precisa ser `online`. Se for `errored` ou `stopped`, tem problema. Ver [10 — Troubleshooting](10-TROUBLESHOOTING.md).

### Ver logs de um processo
```bash
# Últimas 30 linhas (saída + erro)
pm2 logs dros-hub --lines 30 --nostream

# Só logs de erro
pm2 logs dros-hub --lines 30 --nostream --err

# Tempo real (Ctrl+C pra sair)
pm2 logs dros-hub --lines 0
```

### Limpar logs antigos (libera disco)
```bash
pm2 flush dros-hub
```

### Reiniciar processo
```bash
# Preferido (mais confiável)
pm2 stop dros-hub && pm2 start dros-hub

# Alternativa (atualiza env junto)
pm2 restart dros-hub --update-env
```

### Detalhes de um processo
```bash
pm2 show dros-hub
```
Mostra path, uptime, restarts, env vars, etc.

---

## Backup do DB

### Backup manual (faça antes de mexer em coisa crítica)
```bash
cp /opt/platform/agency-hub/server/data/hub.db \
   /opt/platform/agency-hub/server/data/hub.db.backup-$(date +%Y%m%d-%H%M)
```

Cria arquivo tipo `hub.db.backup-20260605-1530`.

### Listar backups
```bash
ls -lh /opt/platform/agency-hub/server/data/*.backup*
```

### Restaurar de backup
```bash
pm2 stop dros-hub
cp /opt/platform/agency-hub/server/data/hub.db.backup-XXX \
   /opt/platform/agency-hub/server/data/hub.db
pm2 start dros-hub
```

> ⚠️ Restaurar **descarta** mudanças desde o backup. Use só quando o DB tá corrompido ou você fez SQL destrutivo por engano.

### Backups antigos (limpar)
```bash
# Lista backups mais velhos que 30 dias
find /opt/platform/agency-hub/server/data/ -name "*.backup*" -mtime +30

# Apaga (cuidado, irreversível)
find /opt/platform/agency-hub/server/data/ -name "*.backup*" -mtime +30 -delete
```

---

## Consultar o DB direto

### Comando básico
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db "SQL_AQUI"
```

### Exemplos úteis

**Listar clientes ativos:**
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db \
  "SELECT id, name, slug FROM clients WHERE is_active=1 ORDER BY name;"
```

**Ver tokens configurados (sem expor valor):**
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db \
  "SELECT key, length(value) as len, substr(value, -4) as last4, updated_at FROM app_settings;"
```

**Contar tarefas por etapa:**
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db \
  "SELECT stage, COUNT(*) FROM tasks WHERE is_active=1 GROUP BY stage;"
```

**Listar usuários:**
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db \
  "SELECT id, name, email, role FROM users WHERE is_active=1;"
```

### Modo interativo
Pra fazer várias queries:
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db
```

Depois digita SQL livre:
```sql
.headers on
.mode column
SELECT * FROM clients LIMIT 5;
.exit
```

---

## Renovar Token Meta (Configurações)

> 📌 **Fácil agora.** Antes era complicado (SSH, nano no .env). Hoje é UI.

### Via UI (recomendado)
1. Login no Hub como dono
2. Sidebar → Configurações
3. Aba **Tokens / Integrações**
4. Seção "Meta / Instagram" → linha "Meta Access Token"
5. Clica em **Trocar** → cola o token novo → **Salvar**

Token novo entra em uso imediatamente (não precisa restart). /core sincroniza nos próximos 10min.

### Como obter token novo
A própria UI tem botão "Como obter" com passo a passo. Resumo:
1. Graph API Explorer (developers.facebook.com/tools/explorer)
2. Selecionar app "Cloude_app_DROS"
3. Obter token de usuário com permissões: `ads_read`, `ads_management`, `business_management`, `pages_show_list`, `pages_read_engagement`, `instagram_basic`
4. Token gerado é curto (1-2h). Estender pra 60 dias via curl:
```bash
curl "https://graph.facebook.com/v21.0/oauth/access_token?grant_type=fb_exchange_token&client_id=944656904876695&client_secret=<SECRET>&fb_exchange_token=<TOKEN_CURTO>"
```
5. Cola o `access_token` retornado na UI

> 💡 **Alternativa permanente:** System User Token no Business Manager (não vence). Veja [Receita 7](12-RECEITAS-PASSO-A-PASSO.md#receita-7).

---

## Checar disco

```bash
df -h
```

Coluna `Use%` na linha `/dev/sda1`:
- < 70%: OK
- 70-85%: monitorar
- > 85%: **agir**. Geralmente Docker logs ou backups antigos.

### Identificar o que tá ocupando
```bash
# Top 10 maiores diretórios da raiz
du -sh /* 2>/dev/null | sort -rh | head -10
```

### Limpar logs do Docker (Evolution costuma encher)
```bash
docker system prune -af --volumes
```

### Limpar logs antigos do PM2
```bash
pm2 flush
```

### Limpar backups antigos
```bash
find / -name "*.backup*" -mtime +60 -size +10M
```
(revisa antes de deletar)

---

## Variáveis de ambiente (.env)

### Ver o .env atual sem expor valores
```bash
cat /opt/platform/agency-hub/.env | sed 's/=.*/=<oculto>/'
```

### Editar .env
```bash
nano /opt/platform/agency-hub/.env
```

> ⚠️ Cuidado com nano. Já tivemos bug de `==` duplo quebrando token. Sempre confere `cat` depois de salvar.

### Aplicar mudanças no .env
```bash
pm2 stop dros-hub && pm2 start dros-hub
```

> 📌 Hoje tokens vêm do DB (via aba Configurações). `.env` é só fallback.

---

## Health check rápido (de manhã)

Cole esse bloco pra verificar tudo de uma vez:
```bash
echo "=== PM2 status ===" && pm2 status
echo "=== Disco ===" && df -h | grep -E "Filesystem|/dev/sda1"
echo "=== Hub erros (1h) ===" && pm2 logs dros-hub --lines 50 --nostream --err 2>&1 | grep -i "error\|exception" | tail -5
echo "=== /core erros (1h) ===" && pm2 logs dros-core --lines 50 --nostream --err 2>&1 | grep -i "error\|exception" | tail -5
echo "=== CRM erros (1h) ===" && pm2 logs dros-crm --lines 50 --nostream --err 2>&1 | grep -i "error\|exception" | tail -5
```

Saída esperada: todos `online`, disco < 85%, sem erros recentes.

---

## Sistema de notificações

Hub usa **SSE (Server-Sent Events)** pra notificações em tempo real. Sino com contagem aparece na sidebar.

Se sino parou de atualizar:
1. Hard refresh no navegador (Ctrl+Shift+R)
2. Se persiste: `pm2 stop dros-hub && pm2 start dros-hub`

---

## Apps Script (sites de venda)

Cada site de venda tem um Apps Script que:
1. Recebe POST do form
2. Salva linha numa Google Sheet
3. Envia pro CRM (via webhook)

### Editar Apps Script
1. Abre a planilha no Drive (ex: "ENTRADA DE LEADS TELHABRAS")
2. Menu → Extensões → Apps Script
3. Edita
4. Salva (Ctrl+S)
5. Pra triggers: `Triggers` no menu lateral

### Logs Apps Script
1. No editor Apps Script: View → Logs
2. Ou: View → Executions

### Trigger não disparou
1. Verificar Triggers (menu lateral) — frequência: a cada 5min
2. Re-instalar: `instalarTrigger()` na função (botão Run)

---

## Capturar logs detalhados de uma requisição

Pra debug pontual:
```bash
# Pega últimas 200 linhas de OUT + ERR
pm2 logs dros-hub --lines 200 --nostream > /tmp/hub-logs.txt

# Filtra por palavra-chave
grep -i "performance\|meta" /tmp/hub-logs.txt | tail -50
```

---

## Comandos perigosos (use com cabeça)

| Comando | O que faz | Risco |
|---|---|---|
| `pm2 delete <name>` | Remove processo da lista | Médio — perde config |
| `rm -rf` | Apaga arquivos recursivamente | Alto — irreversível |
| `git push --force` | Sobrescreve histórico remoto | Alto — perde commits |
| `DROP TABLE X` | Apaga tabela do DB | Alto — sem backup, perde tudo |
| `docker system prune -af --volumes` | Apaga TODOS containers/volumes não usados | Médio — vai precisar reconstruir |
| `pm2 kill` | Mata o daemon do PM2 | Alto — tudo cai |

> 📌 Antes de qualquer um desses: **respira, pensa, faz backup**.

---

## Próximo passo

Próximos arquivos são catálogos das funcionalidades de cada sistema:
- [07 — Funcionalidades do Hub](07-FUNCIONALIDADES-HUB.md)
- [08 — Funcionalidades do /core](08-FUNCIONALIDADES-CORE.md)
- [09 — Funcionalidades do CRM](09-FUNCIONALIDADES-CRM.md)
