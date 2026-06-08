# 08 — Funcionalidades do /core

> Painel de Performance standalone. Roda em paralelo ao Hub, mesmas APIs externas.

URL: https://drosagencia.com.br/core
Repo: `client-dashboard/`
Processo PM2: `dros-core` (porta 3004)

---

## Pra que serve

O `/core` é a versão **standalone** do painel de performance. Foi construído primeiro, antes do Hub. Hoje continua existindo porque:
- Algumas pessoas/clientes ainda usam essa URL direta
- O Hub usa o `/core` em **modo embed** (iframe) pra mostrar dashboards de cliente

Eventualmente vai ser descontinuado — toda a funcionalidade já está dentro do Hub.

> 📌 Mesmas APIs externas que o Hub: Meta, IG, Google Ads, GA4, Kiwify.

---

## Login

```
URL: drosagencia.com.br/core
Email: admin@drosagencia.com.br
Senha: dros2026
```

Único usuário admin. Sem multi-tenancy de usuários (diferente do Hub que tem dono/gerente/etc).

---

## Lista de contas (sidebar)

Quando você entra logado, o sidebar lista todas as **contas Meta Ads** que o token tem acesso, **filtradas** por:

1. **`ALLOWED_CLIENTS`** — lista hardcoded de substrings (no `client-dashboard/server/index.js:39-59`)
2. **`HUB_CLIENT_NAMES`** — lista dinâmica vinda do Hub (sync a cada 10min via `/api/config/clients`)

A função `isAllowedAccount(name)` checa **ambas as listas** (OR). Se o nome da conta Meta contém alguma das substrings, aparece no sidebar.

### Lista hardcoded atual (parcial)
```js
const ALLOWED_CLIENTS = [
  'quimiprol', 'ask equipamentos',
  "d'avila", 'door grill',
  'daiana', 'renove', 'sameco',
  'josi terapeuta', 'bg imob',
  'autocar', 'fernando correa',
  'kellermann', 'ludus', 'essenza',
  'mb vidros', 'agrozacca', 'emdi',
  'oxi dpr', 'dpr',
  'box paper', 'ambiental higiene', 'gringa',
  'telhabras', 'telha.bras',
]
```

### Sync do Hub
A cada 10min, função `syncFromHub()` busca:
- `GET /api/config/tokens` (com header `X-Core-Secret: <secret>`) → atualiza `META_TOKEN`, `GOOGLE_ADS_*`
- `GET /api/config/clients` → atualiza `HUB_CLIENT_NAMES`

Se Hub estiver offline, mantém valores do `.env` (fallback).

---

## Views por conta

Depois de selecionar uma conta no sidebar, abre o dashboard. Tabs no topo:

### 1. Overview
Cards-resumo com todas as métricas:
- **Meta Ads** — gasto, impressões, cliques, conversões
- **Instagram** — seguidores, alcance, engajamento
- **Google Ads** — gasto, cliques, conversões (se conta vinculada)
- **GA4** — sessões, usuários, conversões (se property vinculada)
- **CRM** — leads recebidos, atribuídos, convertidos
- **Kiwify** — vendas, receita (se cliente é Kiwify)

### 2. Meta Ads
Detalhado: campanhas, ads, criativos, performance por idade/gênero/local.

### 3. Instagram
Métricas da página IG: posts, stories, alcance.

### 4. Google Ads
Campanhas, ads, palavras-chave, performance por dispositivo/hora/conversão.

### 5. Analytics (GA4)
Sessões por canal, páginas mais vistas, conversões.

### 6. CRM
Funil de leads daquele cliente (dados do `dros-crm`).

### 7. Kiwify
Vendas e produtos (só se cliente é Kiwify).

`[INSERIR PRINT: dashboard de uma conta no /core]`

---

## Modo Embed (iframe no Hub)

Quando o Hub embuti o /core num iframe pra mostrar dashboard pro cliente, ele:

1. Mint um JWT assinado com `CORE_EMBED_SECRET`
2. Coloca na URL: `/core/?embed_token=<jwt>&account=<nome>`
3. /core valida o JWT no middleware `auth` (linha ~80 de `server/index.js`)
4. Se válido, libera acesso sem precisar de login

### Vantagens
- Cliente vê dashboard no Hub sem precisar fazer login no /core
- Filtra automaticamente pra ver só a conta dele

### Risco
Se `CORE_EMBED_SECRET` vazar, qualquer um pode mint um JWT válido. Por isso é mantido só no `.env` da VPS (não vai pro DB, não vai pro git).

---

## Stack técnica

| Aspecto | Detalhe |
|---|---|
| Node | 16.x via nvm |
| Express | 5.x (precisa `/core/{*path}` em vez de `/core/*`) |
| Frontend | React + Vite (mais simples que o Hub) |
| Auth | JWT (próprio) + JWT embed (compartilhado com Hub) |
| Cache token Kiwify | 90h (refresh aos 90h, expira 96h) |
| Cache token GAds | Refresh a cada 50min (token vive 1h) |
| Cache CRM | em memória, 5min |

---

## Endpoints principais

| Endpoint | O que retorna |
|---|---|
| `POST /api/auth/login` | JWT após login admin |
| `GET /api/meta/accounts` | Lista contas Meta filtradas |
| `GET /api/meta/accounts/:id/insights/compare` | Métricas Meta com comparação período |
| `GET /api/instagram/accounts` | Contas IG vinculadas a páginas FB |
| `GET /api/google-ads/accounts` | Contas Google Ads do MCC |
| `GET /api/analytics/properties` | Properties GA4 acessíveis |
| `GET /api/analytics/:propertyId/report` | Report GA4 |
| `GET /api/crm/:accountId` | Funil de leads desse cliente |
| `GET /api/kiwify/sales` | Vendas Kiwify |
| `GET /api/overview/:accountId` | Overview agregado de tudo |

---

## Quando você vai mexer

- Bug específico de uma view (raro — feature freeze, foco no Hub)
- Adicionar novo cliente: só `ALLOWED_CLIENTS` ou cadastrar no Hub (sync pega em 10min)
- Renovar token: hoje vem do Hub automaticamente — só atualiza lá

### Quase nunca
- Mudar layout
- Adicionar nova métrica (faz no Hub primeiro)
- Migrar pra outro framework

---

## Deploy do /core

```bash
# Local
cd c:\Users\usuar\Downloads\Open Squad\client-dashboard
npm run build
cd ..
git add client-dashboard/
git commit -m "feat(core): ..."
git push core master

# VPS
ssh root@vps-...
cd /root/core
git pull origin master
pm2 stop dros-core && pm2 start dros-core
```

---

## .env do /core

Cuidado: o /core lê de **`/root/core/.env`** (não `client-dashboard/.env`).

Isso é porque o `server/index.js:18`:
```js
dotenv.config({ path: resolve(__dirname, '../../.env') })
```

`__dirname` = `/root/core/client-dashboard/server`, daí `../../.env` = `/root/core/.env`.

### Variáveis principais
```
META_ACCESS_TOKEN=...     # fallback se Hub offline
GOOGLE_ADS_*=...          # idem
KIWIFY_CLIENT_ID=...      # /core lê só do .env (não sync do Hub)
KIWIFY_CLIENT_SECRET=...
KIWIFY_ACCOUNT_ID=...
JWT_SECRET=...
CORE_EMBED_SECRET=...     # mesma chave do Hub
HUB_URL=http://localhost:3003  # opcional, default já é isso
```

---

## Próximo passo

[09 — Funcionalidades do CRM](09-FUNCIONALIDADES-CRM.md).
