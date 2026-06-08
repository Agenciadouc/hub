# 12 — Receitas Passo a Passo

> Cookbook operacional. Cada receita é independente. Copia, segue os passos, valida.

---

## Índice rápido

### A — Setup inicial
- [Receita 1 — Instalar Claude Code](#receita-1)
- [Receita 2 — Clonar o repo Open Squad](#receita-2)
- [Receita 3 — Adicionar os 3 remotes Git](#receita-3)
- [Receita 4 — Configurar SSH pra VPS](#receita-4)
- [Receita 5 — Primeiro pm2 status na VPS](#receita-5)
- [Receita 6 — Logar no Hub e /core pela primeira vez](#receita-6)

### B — Manutenção operacional
- [Receita 7 — Renovar Token Meta via UI](#receita-7)
- [Receita 8 — Gerar System User Token Meta (permanente)](#receita-8)
- [Receita 9 — Backup do DB do Hub](#receita-9)
- [Receita 10 — Verificar disco e limpar](#receita-10)
- [Receita 11 — Atualizar lista de clientes do Painel](#receita-11)
- [Receita 12 — Arquivar cliente](#receita-12)

### C — Diagnóstico
- [Receita 13 — "O Hub não responde"](#receita-13)
- [Receita 14 — "O /core mostra sidebar vazio"](#receita-14)
- [Receita 15 — "Performance mostra Sem dados"](#receita-15)
- [Receita 16 — "CRM não recebe leads"](#receita-16)
- [Receita 17 — "Disco cheio"](#receita-17)
- [Receita 18 — Ler logs do PM2 (interpretar)](#receita-18)

### D — Mudar código (com Claude)
- [Receita 19 — Abrir Claude no projeto](#receita-19)
- [Receita 20 — Template de prompt bom pra feature nova](#receita-20)
- [Receita 21 — Checklist pra revisar plano do Claude](#receita-21)
- [Receita 22 — Acompanhar implementação fase por fase](#receita-22)
- [Receita 23 — Validar antes de subir pra produção](#receita-23)
- [Receita 24 — Subir mudança pra produção (Hub)](#receita-24)
- [Receita 25 — Rollback se algo quebrou](#receita-25)
- [Receita 26 — Atualizar a doc no mesmo commit](#receita-26)

### E — Operações específicas
- [Receita 27 — Criar uma tarefa recorrente](#receita-27)
- [Receita 28 — Criar uma tarefa mãe com subtarefas](#receita-28)
- [Receita 29 — Linha editorial automática](#receita-29)
- [Receita 30 — Aprovar carrossel de aprovação](#receita-30)
- [Receita 31 — Cadastrar cliente novo no Hub](#receita-31)
- [Receita 32 — Vincular IDs Meta/IG/GAds/GA4 a cliente existente](#receita-32)
- [Receita 33 — Exportar tarefas pra Excel](#receita-33)
- [Receita 34 — Gerar token de aprovação pública](#receita-34)
- [Receita 35 — Configurar timer automático de produção](#receita-35)

### F — DB e dados sensíveis
- [Receita 36 — Apagar tarefa de teste do DB](#receita-36)
- [Receita 37 — Resetar senha de um usuário](#receita-37)
- [Receita 38 — Reativar cliente arquivado](#receita-38)
- [Receita 39 — Restaurar DB de backup](#receita-39)
- [Receita 40 — Migração tokens .env → DB](#receita-40)

### G — Casos especiais que já rolaram
- [Receita 41 — Token Meta com == duplo](#receita-41)
- [Receita 42 — Tabela com schema parcial](#receita-42)
- [Receita 43 — Express 5 quebrando /core/*](#receita-43)
- [Receita 44 — PM2 não pega .env novo](#receita-44)

---

## A — Setup inicial

### Receita 1 — Instalar Claude Code <a id="receita-1"></a>

**Quando usar:** primeira vez que vai mexer em código da Dros.
**Pré-requisitos:** Node.js 18+ instalado.
**Tempo:** ~10 min.
**Risco:** baixo.

#### Passo a passo

1. Verifica se tem Node:
   ```bash
   node --version
   ```
   Se não tiver ou for < 18: instala em https://nodejs.org/.

2. Instala Claude Code:
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

3. Abre Claude pela primeira vez:
   ```bash
   claude
   ```

4. Vai abrir o navegador pra login. Loga com a conta Anthropic da Dros.

5. Depois de logar, volta ao terminal. Claude mostra prompt.

#### Validação
Roda `claude --version` — retorna a versão.

#### Se algo deu errado
- Permissão negada no Windows: roda terminal como administrador
- "Command not found" depois de instalar: reabre o terminal pra recarregar PATH

---

### Receita 2 — Clonar o repo Open Squad <a id="receita-2"></a>

**Quando usar:** primeira vez no PC.
**Pré-requisitos:** Git instalado, acesso ao GitHub da Dros.
**Tempo:** ~5 min (depende da internet).
**Risco:** baixo.

#### Passo a passo

1. Cria a pasta onde vai morar:
   ```bash
   cd c:\Users\usuar\Downloads
   ```

2. Clona o repo principal:
   ```bash
   git clone https://github.com/soaresjoaoluiz1/platform.git "Open Squad"
   cd "Open Squad"
   ```

   > 📌 Por que o nome "Open Squad" e não "platform"? O umbrella vem do framework Opensquad. O repo `platform` é o Hub principal, mas a pasta carrega o nome do umbrella.

#### Validação
```bash
ls
```
Deve listar `agency-hub/`, `client-dashboard/`, `crm-dashboard/`, `docs/`, etc.

---

### Receita 3 — Adicionar os 3 remotes Git <a id="receita-3"></a>

**Quando usar:** depois de clonar, se os remotes não estão configurados.
**Tempo:** ~2 min.
**Risco:** baixo.

#### Passo a passo

1. Verifica remotes atuais:
   ```bash
   git remote -v
   ```

2. Se faltar algum, adiciona:
   ```bash
   git remote add platform https://github.com/soaresjoaoluiz1/platform.git
   git remote add core https://github.com/soaresjoaoluiz1/core.git
   git remote add crm https://github.com/soaresjoaoluiz1/crm.git
   ```

3. Testa conexão (faz fetch sem alterar nada):
   ```bash
   git fetch platform
   git fetch core
   git fetch crm
   ```

#### Validação
`git remote -v` lista os 3 remotes (platform, core, crm) com URLs do GitHub.

---

### Receita 4 — Configurar SSH pra VPS <a id="receita-4"></a>

**Quando usar:** primeira vez que precisa acessar a VPS.
**Pré-requisitos:** chave SSH gerada localmente.
**Tempo:** ~10 min.
**Risco:** baixo.

#### Passo a passo

1. **Se não tem chave SSH ainda:**
   ```bash
   ssh-keygen -t ed25519 -C "seu@email.com"
   ```
   Pressiona Enter pra usar caminho padrão. Define passphrase (ou deixa vazio).

2. Copia a chave pública:
   ```bash
   # Windows
   cat ~/.ssh/id_ed25519.pub
   # Mac/Linux
   pbcopy < ~/.ssh/id_ed25519.pub
   ```

3. Pede ao João Luiz ou ao administrador atual da VPS pra adicionar essa chave em `/root/.ssh/authorized_keys` na VPS.

4. Testa conexão:
   ```bash
   ssh root@vps-5269157.3store.com.br
   ```

#### Validação
Conecta sem pedir senha. Mostra prompt `root@vps-5269157`.

#### Se algo deu errado
- "Permission denied": chave não foi cadastrada na VPS, fala com admin
- "Connection refused": IP errado ou VPS off, confirma com admin

---

### Receita 5 — Primeiro pm2 status na VPS <a id="receita-5"></a>

**Quando usar:** depois de conseguir SSH, pra ver o que tá rodando.
**Tempo:** ~1 min.
**Risco:** zero (read-only).

#### Passo a passo

```bash
ssh root@vps-5269157.3store.com.br
pm2 status
```

#### O que esperar
Tabela com 5+ processos, todos `online`:
- dros-hub
- dros-core
- dros-crm
- dros-oxi-pedidos
- gestao-clin

#### Se algum estiver `stopped` ou `errored`
Ver [Receita 13](#receita-13).

---

### Receita 6 — Logar no Hub e /core pela primeira vez <a id="receita-6"></a>

**Quando usar:** sanity check de acesso.
**Tempo:** ~2 min.
**Risco:** zero.

#### Passo a passo

1. Abre navegador → https://drosagencia.com.br/hub
2. Login com seu usuário (peça ao João Luiz se não tem)
3. Confirma que carrega o Dashboard
4. Abre nova aba → https://drosagencia.com.br/core
5. Login: `admin@drosagencia.com.br` / `dros2026`
6. Confirma que sidebar lista contas

#### Se /core mostra sidebar vazio
Ver [Receita 14](#receita-14).

---

## B — Manutenção operacional

### Receita 7 — Renovar Token Meta via UI <a id="receita-7"></a>

**Quando usar:** token Meta vencido (Performance "Sem dados" ou logs com "Session expired").
**Tempo:** ~15 min.
**Risco:** baixo (UI mostra o que está fazendo).

#### Passo a passo

1. Login no Hub como dono → https://drosagencia.com.br/hub
2. Sidebar → **Configurações**
3. Aba **Tokens / Integrações**
4. Seção "Meta / Instagram" → linha "Meta Access Token"
5. Clica **"Como obter"** pra ver passo a passo dentro da UI
6. Em outra aba, vai pra https://developers.facebook.com/tools/explorer/
7. Topo: selecionar app **"Cloude_app_DROS"**
8. "Usuário ou Página" → **"Obter token de acesso do usuário"**
9. Marca permissões: `ads_read`, `ads_management`, `business_management`, `pages_show_list`, `pages_read_engagement`, `instagram_basic`
10. Clica **Gerar Token Acesso** — copia
11. Estende pra 60 dias (em qualquer terminal):
    ```bash
    curl "https://graph.facebook.com/v21.0/oauth/access_token?grant_type=fb_exchange_token&client_id=944656904876695&client_secret=<SECRET>&fb_exchange_token=<TOKEN_CURTO>"
    ```
    > 📌 SECRET fica em https://developers.facebook.com/apps/944656904876695/settings/basic/ (Mostrar Chave Secreta — pede senha).
12. Resposta vem JSON com `access_token` longo. Copia esse.
13. Volta na UI do Hub → **Trocar** → cola o token longo → **Salvar**

#### Validação
1. Status do campo muda pra "✓ Configurado"
2. Hub → Performance → carrega gráficos sem erro

#### Se ainda mostra "Sem dados"
- Confirma que `META_ACCESS_TOKEN` salvou (refresh + ver "Atualizado em ...")
- Se token tem `==` duplo: ver [Receita 41](#receita-41)

---

### Receita 8 — Gerar System User Token Meta (permanente) <a id="receita-8"></a>

**Quando usar:** cansou de renovar a cada 60 dias.
**Tempo:** ~20 min.
**Risco:** baixo (Token User não substitui — só fica disponível como alternativa).

#### Passo a passo

1. Vai em https://business.facebook.com/settings/system-users
2. Se não tem System User: **Adicionar** → "Admin" (não "Funcionário")
3. Clica no System User → **Gerar novo token**
4. App: seleciona **"Cloude_app_DROS"** (ou cria novo se preferir)
5. Permissões (todas):
   - `ads_read`
   - `ads_management`
   - `business_management`
   - `pages_show_list`
   - `pages_read_engagement`
   - `instagram_basic`
6. Copia o token (esse não vence — System User tokens são permanentes por padrão)
7. Cola na UI: Configurações → Tokens → Meta Access Token → Trocar → Salvar

#### Validação
Mesma da Receita 7. Bonus: nunca mais precisa renovar.

---

### Receita 9 — Backup do DB do Hub <a id="receita-9"></a>

**Quando usar:** antes de qualquer SQL destrutivo ou mudança grande no DB.
**Tempo:** ~30 seg.
**Risco:** zero.

#### Passo a passo

```bash
ssh root@vps-5269157.3store.com.br
cp /opt/platform/agency-hub/server/data/hub.db \
   /opt/platform/agency-hub/server/data/hub.db.backup-$(date +%Y%m%d-%H%M)
```

#### Validação
```bash
ls -lh /opt/platform/agency-hub/server/data/*.backup*
```
Mostra o arquivo criado, geralmente 1-50 MB.

---

### Receita 10 — Verificar disco e limpar <a id="receita-10"></a>

**Quando usar:** alertas de "disco cheio" ou semanalmente como manutenção.
**Tempo:** ~5-10 min.
**Risco:** médio (algumas limpezas são irreversíveis).

#### Passo a passo

1. Checa o uso atual:
   ```bash
   df -h
   ```
   Foca na linha `/dev/sda1`.

2. Identifica o que ocupa:
   ```bash
   du -sh /* 2>/dev/null | sort -rh | head -10
   ```

3. **Se Docker (Evolution) é o vilão:**
   ```bash
   docker system prune -af --volumes
   ```
   > ⚠️ Remove volumes não usados. Confirma se Evolution não vai perder estado (geralmente OK).

4. **Se logs do PM2 estão grandes:**
   ```bash
   pm2 flush
   ```

5. **Se backups antigos:**
   ```bash
   find /opt/platform -name "*.backup*" -mtime +30 -size +10M
   # Verifica antes:
   ls -lh <arquivos retornados acima>
   # Apaga:
   find /opt/platform -name "*.backup*" -mtime +30 -size +10M -delete
   ```

6. **Logs do sistema:**
   ```bash
   journalctl --vacuum-time=7d
   ```

#### Validação
```bash
df -h
```
Uso deve cair pra < 80%.

---

### Receita 11 — Atualizar lista de clientes do Painel <a id="receita-11"></a>

**Quando usar:** novo cliente assinou contrato e precisa aparecer no Hub/Core.
**Tempo:** ~5 min.
**Risco:** baixo.

#### Passo a passo

1. Hub → Configurações → aba **Contas do Painel**
2. **+ Nova Conta**
3. Preencher:
   - Nome (ex: "Box Paper Embalagens")
   - Slug (auto-gerado, confirma)
   - Email de contato
   - **Meta Ads** — dropdown com search → procura a conta do cliente → seleciona
   - **Instagram** — idem
   - **Google Ads** — idem (se cliente tem)
   - **GA4** — idem (se cliente tem)
4. **Criar Conta**

#### Validação
- Aparece na lista da aba Contas
- Hub → Performance → cliente aparece com badges das plataformas
- /core (após 10min de sync) → sidebar lista a conta

#### Se sumiu antes dos 10min
Roda restart do /core pra forçar sync imediato:
```bash
ssh root@vps-... && pm2 stop dros-core && pm2 start dros-core
```

---

### Receita 12 — Arquivar cliente <a id="receita-12"></a>

**Quando usar:** cliente cancelou contrato.
**Tempo:** ~1 min.
**Risco:** baixo (é soft delete — dá pra reativar).

#### Passo a passo

1. Hub → Configurações → aba **Contas do Painel**
2. Encontra o cliente
3. Botão **Arquivar** (ícone caixinha)
4. Confirma

#### Validação
- Cliente some da lista (a menos que marque "Mostrar arquivadas")
- Sumiu do /core após próximo sync (10min)
- Tarefas do cliente permanecem no histórico, mas cliente não aparece em novos cadastros

#### Pra reativar
Ver [Receita 38](#receita-38).

---

## C — Diagnóstico

### Receita 13 — "O Hub não responde" <a id="receita-13"></a>

**Quando usar:** drosagencia.com.br/hub não carrega.
**Tempo:** ~10 min (depende do problema).
**Risco:** baixo (só investigando).

#### Passo a passo

1. Confirma se VPS tá acessível:
   ```bash
   ssh root@vps-5269157.3store.com.br
   ```

2. Status do PM2:
   ```bash
   pm2 status
   ```

3. Se `dros-hub` está `stopped` ou `errored`:
   ```bash
   pm2 logs dros-hub --lines 50 --nostream --err
   ```

4. **Analisa o erro nos logs:**

| Erro | Causa | Solução |
|---|---|---|
| `SyntaxError` | Código com bug | [Receita 25](#receita-25) — rollback |
| `EADDRINUSE` | Porta 3003 em uso | `lsof -i :3003` → `kill <PID>` |
| `Cannot find module` | npm install esquecido | `cd /opt/platform/agency-hub && npm install` |
| `SqliteError: no such column` | Migration faltando | [Receita 42](#receita-42) |
| `ENOSPC` (sem espaço) | Disco cheio | [Receita 10](#receita-10) |

5. Depois de resolver, reinicia:
   ```bash
   pm2 stop dros-hub && pm2 start dros-hub
   ```

#### Validação
- `pm2 status` mostra `online`
- Abre Hub no navegador, Dashboard carrega

---

### Receita 14 — "O /core mostra sidebar vazio" <a id="receita-14"></a>

**Quando usar:** /core sem contas no sidebar.
**Tempo:** ~5 min.
**Risco:** baixo.

#### Passo a passo

1. **Primeiro: cache do navegador**
   - F12 → Network → "Disable cache" → Ctrl+Shift+R
   - Se aparecer contas: era cache.

2. Se não resolveu:
   ```bash
   ssh root@vps-...
   pm2 logs dros-core --lines 30 --nostream
   ```

3. Confirma sync com Hub:
   ```
   [Hub sync] tokens atualizados X chaves
   [Hub sync] clientes recebidos: Y
   ```
   Se não aparece, sync falhou.

4. **Testa API direto:**
   ```bash
   curl -s -H "X-Core-Secret: dros-core-embed-2026-shared-key" \
     "http://localhost:3003/api/config/tokens" | head -c 200
   ```
   Esperado: JSON com tokens. Se 401: secret incorreto. Se conexão recusada: Hub down.

5. **Testa /core API:**
   ```bash
   TOKEN_CORE=$(curl -s -X POST http://localhost:3004/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email":"admin@drosagencia.com.br","password":"dros2026"}' | grep -oP '"token":"[^"]+' | cut -d'"' -f4)
   curl -s -H "Authorization: Bearer $TOKEN_CORE" "http://localhost:3004/api/meta/accounts" | head -c 500
   ```
   Esperado: `{"accounts":[...]}` com contas.

6. Se backend OK mas frontend não — Service Worker:
   - DevTools → Application → Service Workers → Unregister
   - Clear storage → "Clear site data"

#### Validação
- Sidebar lista contas
- Logs mostram sync OK
- API retorna contas

---

### Receita 15 — "Performance mostra Sem dados" <a id="receita-15"></a>

**Quando usar:** cards da Performance todos zerados ou erro.
**Tempo:** ~5 min.
**Risco:** zero.

#### Passo a passo

1. **Token vencido?** Configurações → Tokens → status "✓ Configurado"?
   Se "Não configurado" ou "Fallback .env": renova → [Receita 7](#receita-7).

2. **IDs vinculados?** Configurações → Contas → cliente X tem badges Meta/IG/GAds/GA4 coloridos?
   Se não: edita cliente, vincula IDs ([Receita 32](#receita-32)).

3. **Cliente novo sem dados de fato?** Confere no Meta Ads Manager se há gasto no período.

4. **Logs:**
   ```bash
   pm2 logs dros-hub --lines 30 --nostream | grep -i "performance\|meta"
   ```

#### Validação
Cards mostram números após refresh.

---

### Receita 16 — "CRM não recebe leads" <a id="receita-16"></a>

**Quando usar:** site submeteu form, mas lead não apareceu no CRM.
**Tempo:** ~10-15 min.
**Risco:** baixo.

#### Passo a passo

1. **Lead chegou na Google Sheet?**
   - Abre a planilha do cliente (ex: "ENTRADA DE LEADS TELHABRAS")
   - Aba "FORM SITE" → confirma linha foi adicionada
   - Se não chegou: problema no form do site (frontend) ou no Apps Script `/exec`

2. **Apps Script trigger rodando?**
   - Extensões → Apps Script → menu lateral "Triggers"
   - Confirma `sincronizarLeadsNovos` ativo, a cada 5min
   - Se não tem: roda `instalarTrigger()` (botão Run no editor)

3. **Apps Script consegue chamar CRM?**
   - No editor: View → Executions
   - Última execução do `sincronizarLeadsNovos` deu OK?
   - Se erro 401/403: URL do CRM errada
   - Se erro 500: CRM com bug

4. **CRM rodando?**
   ```bash
   pm2 status | grep crm
   pm2 logs dros-crm --lines 30 --nostream --err
   ```

5. **Evolution rodando?**
   ```bash
   docker ps | grep evolution
   ```
   Se algum container `Exited`: `docker start <container-name>`.
   Se Postgres crashou: muito provável disco cheio → [Receita 10](#receita-10).

#### Validação
Manda lead de teste, em 5min aparece no CRM.

---

### Receita 17 — "Disco cheio" <a id="receita-17"></a>

Ver [Receita 10](#receita-10) — instruções completas.

---

### Receita 18 — Ler logs do PM2 (interpretar) <a id="receita-18"></a>

**Quando usar:** sempre que algo dá errado.
**Tempo:** ~5 min.
**Risco:** zero.

#### Passo a passo

1. **Logs gerais (out + err):**
   ```bash
   pm2 logs dros-hub --lines 30 --nostream
   ```

2. **Só erros:**
   ```bash
   pm2 logs dros-hub --lines 30 --nostream --err
   ```

3. **Tempo real (acompanha enquanto testa algo):**
   ```bash
   pm2 logs dros-hub --lines 0
   ```
   Ctrl+C pra sair.

4. **Filtra:**
   ```bash
   pm2 logs dros-hub --lines 200 --nostream | grep -i "meta\|expired"
   ```

#### Padrões comuns

| Padrão | Significado |
|---|---|
| `Listening on port 3003` | Boot OK |
| `[settings] seed inicial: X chaves` | Seed do `.env` rodou |
| `[Recurring] criadas X tarefa(s)` | Cron de recorrência rodou |
| `Session has expired` | Token Meta vencido |
| `Cannot parse access token` | Token Meta corrompido |
| `SqliteError` | Problema com DB |
| `EADDRINUSE` | Porta em uso |
| `ECONNREFUSED` | Conexão recusada (serviço externo down) |

---

## D — Mudar código (com Claude)

### Receita 19 — Abrir Claude no projeto <a id="receita-19"></a>

```bash
cd c:\Users\usuar\Downloads\Open Squad
claude
```

Sempre na pasta umbrella `Open Squad` (não em subpastas). Claude vê o repo todo.

---

### Receita 20 — Template de prompt bom pra feature nova <a id="receita-20"></a>

**Quando usar:** vai pedir feature nova ao Claude.
**Tempo:** ~3 min escrevendo prompt.

#### Estrutura

```
SISTEMA: <Hub / /core / CRM>

PROBLEMA: <1 frase descrevendo o sintoma ou necessidade>

O QUE QUERO: <3-5 frases descrevendo o resultado esperado>

ONDE: <página/rota/arquivo específico, se souber>

NÃO QUERO: <coisas a evitar — opcional>

VAMOS EM PLAN MODE PRIMEIRO.
```

#### Exemplo real

```
SISTEMA: Hub

PROBLEMA: Hoje tokens Meta/Google ficam no .env da VPS. Vence a cada 60 dias e preciso SSH na VPS pra trocar.

O QUE QUERO: Uma aba "Configurações" no sidebar (dono only) com:
- Lista de tokens (Meta, Google Ads etc) com input mascarado tipo senha
- Eye/EyeOff pra revelar quando editando
- Botão "Como obter" com passo a passo
- Salvar sem precisar restart

NÃO QUERO: criar nova tabela complexa. Idealmente reaproveita o que tem.

VAMOS EM PLAN MODE PRIMEIRO.
```

---

### Receita 21 — Checklist pra revisar plano do Claude <a id="receita-21"></a>

Quando Claude termina o Plan Mode, antes de aprovar:

- [ ] Entendi o contexto descrito? Bate com o que pedi?
- [ ] Os arquivos listados pra modificar fazem sentido?
- [ ] Tem fases claras? Cada fase deixa sistema utilizável?
- [ ] Tem seção de "edge cases" ou "riscos"? Lista cenários esperados?
- [ ] Tem verificação end-to-end clara?
- [ ] Sequência de commits faz sentido (cada commit é coerente)?
- [ ] O escopo está claro — sei o que NÃO está incluído?

Se algo não bate, peça ajuste em vez de aprovar.

---

### Receita 22 — Acompanhar implementação fase por fase <a id="receita-22"></a>

Quando Claude começa a implementar (depois de aprovar):

1. **Olhe a tela.** Claude mostra cada arquivo modificado em tempo real.
2. **Não fala "ok" automático.** Se Claude perguntar algo, leia antes de responder.
3. **Diff:** se a mudança é grande, pede `git diff <arquivo>` pra ver o que mudou.
4. **Build:** quando Claude rodar `npm run build`, confere se passou sem erro.
5. **Sanity check:** abre o navegador no localhost (se aplicável) ou pede pra Claude mostrar exemplo de uso.

---

### Receita 23 — Validar antes de subir pra produção <a id="receita-23"></a>

Checklist pré-deploy:

- [ ] `npm run build` rodou sem erro
- [ ] Confere `git status` — arquivos esperados modificados, nada estranho
- [ ] Confere `git diff` — mudanças fazem sentido
- [ ] (Se mexeu DB) Backup feito ([Receita 9](#receita-9))
- [ ] (Se mudou frontend) `dist/` foi atualizado e tá no `git add`
- [ ] Mensagem do commit segue padrão `feat(escopo): descrição`

---

### Receita 24 — Subir mudança pra produção (Hub) <a id="receita-24"></a>

**Quando usar:** depois de aprovar feature + build local OK.
**Tempo:** ~3-5 min.
**Risco:** médio (se quebrar, ver [Receita 25](#receita-25)).

#### Passo a passo

1. Confirma build local:
   ```bash
   cd c:\Users\usuar\Downloads\Open Squad\agency-hub
   npm run build
   ```

2. Commit + push:
   ```bash
   cd ..
   git add agency-hub/
   git commit -m "feat(agency-hub): <descrição>"
   git push platform master
   ```

3. SSH na VPS:
   ```bash
   ssh root@vps-...
   ```

4. Pull + restart:
   ```bash
   cd /opt/platform
   git pull origin master
   pm2 stop dros-hub && pm2 start dros-hub
   ```

5. Sanity:
   ```bash
   sleep 10 && pm2 logs dros-hub --lines 20 --nostream --err
   ```

6. Abre Hub no navegador (Ctrl+Shift+R) e testa a feature.

#### Validação
- Logs sem erro
- Feature funciona

#### Se algo deu errado
[Receita 25](#receita-25) — rollback.

---

### Receita 25 — Rollback se algo quebrou <a id="receita-25"></a>

**Quando usar:** acabou de deployar e produção tá quebrada.
**Tempo:** ~3 min.
**Risco:** baixo se feito rápido.

#### Passo a passo

1. **Local:**
   ```bash
   git revert HEAD
   git push platform master
   ```

2. **VPS:**
   ```bash
   ssh root@vps-...
   cd /opt/platform
   git pull origin master
   pm2 stop dros-hub && pm2 start dros-hub
   ```

3. **Se mexeu DB e fez backup:**
   ```bash
   pm2 stop dros-hub
   cp /opt/platform/agency-hub/server/data/hub.db.backup-XXX \
      /opt/platform/agency-hub/server/data/hub.db
   pm2 start dros-hub
   ```

#### Validação
Sistema voltou ao estado anterior. Testa.

---

### Receita 26 — Atualizar a doc no mesmo commit <a id="receita-26"></a>

Política Dros: features importantes atualizam a doc no MESMO commit.

#### Passo a passo

1. Implementa a feature.
2. **Antes do commit:** abre os arquivos relevantes em `docs/`:
   - Adicionou rota nova? → atualiza `07-FUNCIONALIDADES-HUB.md`
   - Adicionou comando? → atualiza `06-OPERACAO-DIARIA.md` ou `12-RECEITAS-PASSO-A-PASSO.md`
   - Mudou workflow? → atualiza `04-NOSSO-WORKFLOW-DEV.md`
3. Adiciona docs no `git add`:
   ```bash
   git add agency-hub/ docs/
   ```
4. Commit único:
   ```bash
   git commit -m "feat(agency-hub): <descrição> (+docs)"
   ```

---

## E — Operações específicas

### Receita 27 — Criar uma tarefa recorrente <a id="receita-27"></a>

**Quando usar:** quer uma tarefa que aparece automaticamente toda semana/mês.
**Tempo:** ~5 min.

#### Passo a passo

1. Hub → Pipeline (ou Tarefas)
2. Botão **Recorrência** (ícone de loop)
3. Modal abre. Preenche:
   - **Nome do template** (interno, ex: "Relatório Mensal ASK")
   - **Tipo:** Normal ou Mãe
   - **Frequência:** Semanal ou Mensal
   - **Dia:** segunda/terça/etc OU dia do mês (1-31)
   - **Hora:** default 06:00
   - **Prazo:** +N dias após criação (ex: 7 = tarefa criada hoje vence em 7 dias)
   - **Cliente, depto, responsáveis, categoria, prioridade, drive links, etc** — como tarefa normal
4. Se Tipo=Mãe: adiciona subtarefas no bloco de baixo
5. **Criar template**

#### Validação
- Aba "Tarefas Recorrentes" (acessível por URL `/hub/tarefas-recorrentes`) lista o template
- Em até 5min do horário programado, tarefa real é criada

#### Pra testar agora
Em `/hub/tarefas-recorrentes`, botão "▶ Executar agora" cria a tarefa imediatamente.

---

### Receita 28 — Criar uma tarefa mãe com subtarefas <a id="receita-28"></a>

**Quando usar:** projeto que tem várias etapas/entregas.
**Tempo:** ~5-10 min.

#### Passo a passo

1. Hub → Pipeline ou Tarefas
2. Botão **Tarefa Mãe**
3. Modal abre. Preenche dados da mãe.
4. No final do modal, bloco "Subtarefas":
   - **+ Adicionar subtarefa** pra cada uma
   - Pra cada sub: título, prazo offset, responsáveis, depto, conteúdo de aprovação
5. **Criar Tarefa Mãe**

#### Comportamento
- Mãe começa em "A fazer"
- Subtarefas começam em "A fazer" também
- Quando todas as subs concluem, mãe auto-fecha

---

### Receita 29 — Linha editorial automática <a id="receita-29"></a>

**Quando usar:** cliente de social media — workflow padrão.
**Tempo:** ~2 min.

#### Passo a passo

1. Hub → Pipeline → botão **Linha Editorial** (dono only)
2. Preenche título e cliente
3. Confirma

#### Comportamento
Mãe criada com 5 subtarefas pré-configuradas:
1. Briefing (Ivandro)
2. Reunião Aprovação Cliente do Briefing
3. Aprovação Interna Final
4. Aprovação Cliente Final
5. Publicação (Graziele)

Conforme você move as subs entre etapas, **triggers automáticos** criam outras subs:
- Briefing → Criar Imagens (Dalila)
- Reunião → Gravação (Ivandro)
- Gravação → Subir Arquivos → Editar Vídeos → Programar Publ Vídeos

Quando todas as 11 subs concluem, mãe fecha.

---

### Receita 30 — Aprovar carrossel de aprovação <a id="receita-30"></a>

**Quando usar:** cliente vai aprovar conteúdo via Hub.
**Tempo:** ~2 min.

#### Passo a passo

**Pra time interno (preparar):**
1. Tarefa em `revisao_interna` → adiciona conteúdo de aprovação
   - Cliente único? `approval_link` simples
   - Carrossel? Marca checkbox + adiciona N links (pode ser Drive)
   - Texto da legenda? `approval_text`
2. Move pra `aprovacao_interna`
3. Gerente aprova internamente → move pra `aguardando_cliente`

**Pra cliente:**
1. Cliente entra em Hub → Aprovações
2. Vê o conteúdo (carrossel renderiza com swipe horizontal)
3. **Aprovar** → vai pra `aprovado_cliente`
4. **OU Solicitar Alteração** → escreve o que mudar → volta pra `revisao_interna` com banner

---

### Receita 31 — Cadastrar cliente novo no Hub <a id="receita-31"></a>

**Quando usar:** novo cliente assinou.
**Tempo:** ~10 min.

#### Passo a passo

1. Hub → Clientes → **+ Novo Cliente**
2. Preenche em seções:
   - **Identificação:** nome, slug (auto), logo
   - **Contato:** nome, email, telefone
   - **Redes/Site:** Instagram, website
   - **Dados fiscais:** CNPJ, razão social, cidade/estado
   - **Contrato/Financeiro:** mensalidade, dia de pagamento, início do contrato
   - **Vínculos do Painel:** 4 dropdowns (Meta/IG/GAds/GA4) — search direto da API
   - **Acesso:** email + senha pro login do cliente no portal
3. **Salvar**

#### Validação
- Cliente aparece em /clients (ativos)
- Performance lista o cliente
- /core sincroniza em 10min

---

### Receita 32 — Vincular IDs Meta/IG/GAds/GA4 a cliente existente <a id="receita-32"></a>

**Quando usar:** cliente já tem cadastro básico mas faltam vínculos do painel.
**Tempo:** ~3 min.

#### Passo a passo

1. Hub → Clientes → clica no cliente
2. Aba **Vínculos do Painel de Performance**
3. Pra cada plataforma, dropdown com search:
   - Digita parte do nome (ex: "box paper")
   - Seleciona a opção que aparece
4. **Salvar**

#### Alternativa rápida (via aba Configurações)
1. Configurações → Contas do Painel
2. Encontra o cliente → ícone Edit (lápis)
3. Mesmo formulário, mais focado

---

### Receita 33 — Exportar tarefas pra Excel <a id="receita-33"></a>

**Quando usar:** análise externa, reunião com cliente.
**Tempo:** ~1 min.

#### Passo a passo

1. Hub → Tarefas
2. Aplica filtros desejados (cliente, período, etapa)
3. Botão **Exportar** (dono only)
4. Download de .xlsx

---

### Receita 34 — Gerar token de aprovação pública <a id="receita-34"></a>

**Quando usar:** cliente quer aprovar sem ter login no Hub.
**Tempo:** ~30 seg.

#### Passo a passo

1. Hub → Clientes → cliente → seção "Aprovação Externa"
2. Botão **Gerar Token** (ou Regenerar)
3. Copia o link gerado: `https://drosagencia.com.br/approvals/<token>`
4. Manda pro cliente

Cliente acessa o link sem login e aprova tarefas em `aguardando_cliente`.

#### Pra revogar
Botão **Revogar Token** no mesmo lugar.

---

### Receita 35 — Configurar timer automático de produção <a id="receita-35"></a>

Não precisa configurar — já é automático.

#### Como funciona
- Tarefa entra em `em_producao` → timer começa pra responsável principal
- Tarefa sai de `em_producao` → timer pausa, sessão salva
- Cron 1h: se timer rodando há +1h, pergunta "ainda em produção?"

#### Manualmente
TaskDetail → seção Tempo → botões **Iniciar** / **Pausar**. Pode forçar timer mesmo fora de `em_producao`.

---

## F — DB e dados sensíveis

### Receita 36 — Apagar tarefa de teste do DB <a id="receita-36"></a>

**Quando usar:** criou tarefa de teste que precisa sumir totalmente (não só soft delete).
**Pré-requisitos:** ID da tarefa.
**Tempo:** ~3 min.
**Risco:** alto — irreversível sem backup.

#### Passo a passo

1. **Backup primeiro!** [Receita 9](#receita-9).

2. Identifica a tarefa:
   ```bash
   sqlite3 /opt/platform/agency-hub/server/data/hub.db \
     "SELECT id, title, client_id FROM tasks WHERE id = 656;"
   ```

3. Apaga em **ordem** (foreign keys):
   ```bash
   sqlite3 /opt/platform/agency-hub/server/data/hub.db <<'SQL'
   DELETE FROM task_history WHERE task_id = 656;
   DELETE FROM task_assignees WHERE task_id = 656;
   DELETE FROM task_comments WHERE task_id = 656;
   DELETE FROM task_attachments WHERE task_id = 656;
   DELETE FROM time_entries WHERE task_id = 656;
   DELETE FROM tasks WHERE parent_task_id = 656;
   DELETE FROM tasks WHERE id = 656;
   SQL
   ```

#### Validação
```bash
sqlite3 .../hub.db "SELECT * FROM tasks WHERE id = 656;"
```
Sem retorno = apagada.

---

### Receita 37 — Resetar senha de um usuário <a id="receita-37"></a>

**Quando usar:** funcionário esqueceu a senha.
**Tempo:** ~2 min.
**Risco:** baixo.

#### Passo a passo

1. **Backup:** [Receita 9](#receita-9).

2. Gera hash da nova senha (precisa Node + bcryptjs disponíveis na VPS):
   ```bash
   cd /opt/platform/agency-hub
   node -e "console.log(require('bcryptjs').hashSync('NOVA_SENHA_AQUI', 10))"
   ```
   Copia o hash retornado.

3. Atualiza no DB:
   ```bash
   sqlite3 server/data/hub.db \
     "UPDATE users SET password = 'HASH_AQUI' WHERE email = 'usuario@email.com';"
   ```

4. Avisa o usuário pra fazer login com a nova senha.

#### Validação
Login com nova senha funciona.

---

### Receita 38 — Reativar cliente arquivado <a id="receita-38"></a>

**Quando usar:** cliente cancelou e voltou.
**Tempo:** ~1 min.

#### Passo a passo

1. Hub → Configurações → Contas do Painel
2. Marca "Mostrar arquivadas"
3. Encontra o cliente
4. Botão **Reativar** (ícone caixinha aberta)

#### Validação
- Cliente volta pra lista de ativos
- Aparece no Performance + /core (após sync)

---

### Receita 39 — Restaurar DB de backup <a id="receita-39"></a>

**Quando usar:** DB corrompido OU SQL destrutivo errado.
**Tempo:** ~3 min.
**Risco:** alto — perde tudo desde o backup.

#### Passo a passo

1. Para o Hub:
   ```bash
   pm2 stop dros-hub
   ```

2. Lista backups:
   ```bash
   ls -lh /opt/platform/agency-hub/server/data/*.backup*
   ```

3. Copia o desejado por cima:
   ```bash
   cp /opt/platform/agency-hub/server/data/hub.db.backup-20260605-1530 \
      /opt/platform/agency-hub/server/data/hub.db
   ```

4. Sobe Hub:
   ```bash
   pm2 start dros-hub
   ```

#### Validação
- `pm2 status` → online
- Hub no navegador funciona
- Verifica que dados estão como esperado

---

### Receita 40 — Migração tokens .env → DB <a id="receita-40"></a>

Já é automática. Roda no startup do Hub via `seedSettingsFromEnv()`.

#### Como funciona
1. Hub liga → roda `seedSettingsFromEnv()`
2. Pra cada chave em `KNOWN_KEYS`, se não existe no DB E existe no `.env`:
3. Insere no `app_settings`
4. Idempotente: rodar de novo não duplica

#### Verificar se rodou
```bash
pm2 logs dros-hub --lines 30 --nostream | grep "settings.*seed"
```
Esperado: `[settings] seed inicial: X chaves importadas do .env`.

---

## G — Casos especiais que já rolaram

### Receita 41 — Token Meta com `==` duplo <a id="receita-41"></a>

**Quando usar:** token Meta com erro "Cannot parse access token" e diagnóstico mostra `=` no início.
**Tempo:** ~2 min.

#### Diagnóstico
```bash
TOKEN=$(sqlite3 /opt/platform/agency-hub/server/data/hub.db \
  "SELECT value FROM app_settings WHERE key='META_ACCESS_TOKEN';")
echo "Primeiros 5: ${TOKEN:0:5}"
```

Se mostrar `=EAA...` em vez de `EAA...`: tem o bug.

#### Solução
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db \
  "UPDATE app_settings SET value = substr(value, 2) WHERE key='META_ACCESS_TOKEN' AND substr(value, 1, 1) = '=';"

sed -i 's/^META_ACCESS_TOKEN==/META_ACCESS_TOKEN=/' /opt/platform/agency-hub/.env

TOKEN=$(sqlite3 /opt/platform/agency-hub/server/data/hub.db \
  "SELECT value FROM app_settings WHERE key='META_ACCESS_TOKEN';")
curl -s "https://graph.facebook.com/v21.0/me?access_token=$TOKEN" | head -c 200
```

Esperado: `{"name": "...", "id": "..."}`.

---

### Receita 42 — Tabela com schema parcial <a id="receita-42"></a>

**Quando usar:** erro `SqliteError: no such column: X` em tabela existente.

#### Diagnóstico
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db "PRAGMA table_info(task_templates);"
```
Compara com schema esperado em `agency-hub/server/db.js`.

#### Solução A (recomendada) — auto-fix do código
O Hub já tem detector. Logs no startup:
```
[migration] task_templates com schema incompleto, recriando.
```
Se aparece, mas erro persiste, vai pra Solução B.

#### Solução B — manual
1. Backup ([Receita 9](#receita-9))
2. DROP e recria:
```bash
sqlite3 /opt/platform/agency-hub/server/data/hub.db <<'SQL'
DROP TABLE IF EXISTS task_template_subtask_assignees;
DROP TABLE IF EXISTS task_template_subtasks;
DROP TABLE IF EXISTS task_template_assignees;
DROP TABLE IF EXISTS task_templates;
SQL
```
3. Restart Hub → migrations recriam do schema novo:
```bash
pm2 stop dros-hub && pm2 start dros-hub
```

---

### Receita 43 — Express 5 quebrando /core/* <a id="receita-43"></a>

**Quando usar:** /core dá erro `PathError: Missing parameter name at index 7: /core/*`.

#### Causa
Express 5 mudou a sintaxe de wildcard.

#### Solução
Edita `client-dashboard/server/index.js`:
```js
// Velho (Express 4):
app.get('/core/*', (req, res) => { ... })

// Novo (Express 5):
app.get('/core/{*path}', (req, res) => { ... })
```

Commit + push pro repo `core` + pull na VPS + stop/start.

---

### Receita 44 — PM2 não pega .env novo <a id="receita-44"></a>

**Quando usar:** editou .env, fez `pm2 restart`, valor velho persiste.

#### Solução
Sempre prefira:
```bash
pm2 stop dros-hub && pm2 start dros-hub
```

Ou força:
```bash
pm2 restart dros-hub --update-env
```

> 📌 Memória da Dros: `feedback_pm2_restart_caching.md`.

---

## Fim do cookbook

Achou que falta uma receita? Adiciona aqui e commita. Política Dros: doc viva.
