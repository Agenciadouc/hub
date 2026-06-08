# 02 — O Que a Dros Tem

> Mapa completo dos sistemas, onde rodam, quem usa, e quando você vai mexer neles.

---

## Tabela-resumo

| Sistema | URL | Repo | Path VPS | Processo PM2 | Porta |
|---|---|---|---|---|---|
| **Hub** | drosagencia.com.br/hub | `platform` | `/opt/platform` | `dros-hub` | 3003 |
| **/core** | drosagencia.com.br/core | `core` | `/root/core` | `dros-core` | 3004 |
| **CRM** | drosagencia.com.br/crm | `crm` | `/root/crm` | `dros-crm` | 3002 |
| Box Paper | boxpaper.com.br | `box-paper-site-*` | (estático) | — | — |
| Telhabras | telhabras.com.br | `telhabras-form` | (estático) | — | — |
| Gringa | gringacosmeticos.com.br | `gringa-em-breve*` | (estático) | — | — |
| Dros Sales | drosagencia.com.br/sales | `dros-sales-site-v3` | (estático) | — | — |
| Sites curtos | varios | varios | (estáticos via Cloudflare) | — | — |

**Servidor:** vps-5269157.3store.com.br (HostGator) — Node.js 16.x

> 💡 **PM2** = gerenciador de processos. Mantém os sistemas rodando 24/7. Se cair, ele reinicia sozinho. Ver [Glossário](11-GLOSSARIO.md#pm2).

---

## 🟢 Hub (Sistema Principal)

**URL:** https://drosagencia.com.br/hub
**Roles:** dono / gerente / funcionário / cliente

### O que faz

É o "ERP da agência". Gerencia:
- **Tarefas** com fluxo Kanban (pipeline configurável)
- **Aprovações** internas (gerente) e do cliente (cliente final aprovou ou pediu ajuste)
- **Performance** — gráficos integrados do Meta Ads, Instagram, Google Ads, GA4 por cliente
- **Clientes** — cadastro completo com vínculos das contas de ads
- **Equipe** — funcionários, departamentos, categorias
- **Gravações** — calendário de captação de vídeo
- **Financeiro** — DRE, despesas, receitas
- **Configurações** — etapas do pipeline, tokens, contas do painel
- **Tarefas Recorrentes** — templates que geram tarefas automaticamente (semanal/mensal)

### Stack técnica

- **Frontend:** React 19 + TypeScript + Vite 6
- **Backend:** Express 5 + Node.js 16
- **DB:** SQLite (`server/data/hub.db`)
- **Auth:** JWT (token assinado)
- **Realtime:** SSE (Server-Sent Events) — notificações em tempo real

### Quando você vai mexer

- Adicionar/remover funcionalidades (sempre via Claude Code)
- Cadastrar clientes novos no painel
- Atualizar tokens das integrações (Configurações)
- Modificar etapas do pipeline (Configurações)
- Subir nova versão pra VPS (deploy)

### Captura

`[INSERIR PRINT: tela inicial do Hub logado como dono]`

---

## 🟡 /core (Painel Performance Standalone)

**URL:** https://drosagencia.com.br/core
**Login admin:** admin@drosagencia.com.br / dros2026
**Roles:** admin (compartilhado) + clientes selecionados

### O que faz

Versão standalone do painel de performance. Antes de tudo virar parte do Hub, era aqui que clientes viam os gráficos de Meta Ads, IG, GAds, GA4 etc.

Hoje:
- Funciona em paralelo com o Hub (mesmas APIs)
- Tem o **modo embed** (iframe no Hub via `?embed_token=xxx`)
- Lista de clientes filtrada por `ALLOWED_CLIENTS` (hardcoded) **+** clientes vindos do Hub (sync a cada 10min)
- Tokens vêm do Hub via API (`/api/config/tokens` com header `X-Core-Secret`) — fallback `.env` se Hub estiver offline

### Stack técnica

- **Frontend:** React + Vite (mais simples que o Hub)
- **Backend:** Express + Node.js 16
- **DB:** nenhum (lê tudo de APIs externas + sync do Hub)
- **Auth:** JWT próprio + JWT embed (compartilhado com Hub via `CORE_EMBED_SECRET`)

### Quando você vai mexer

- Adicionar nova métrica ou visualização
- Corrigir bug específico de uma view
- Manutenção de tokens (mas tokens hoje vêm do Hub automaticamente)

### Captura

`[INSERIR PRINT: tela do /core com sidebar de contas + dashboard de uma conta]`

---

## 🔵 CRM (Leads WhatsApp)

**URL:** https://drosagencia.com.br/crm
**Roles:** admin / atendente

### O que faz

Gerenciador de leads vindos do WhatsApp via **Evolution API** (gateway oficial não-oficial 🙃).

- Recebe leads de webhooks dos sites de venda (via Google Apps Script trigger 5min)
- Atribui automaticamente a atendentes (humanos ou agentes IA)
- Conversa via interface tipo WhatsApp Web
- Follow-ups automáticos (mensagens agendadas se lead não responde)
- Métricas por atendente (taxa de resposta, conversão, tempo médio)

### Stack técnica

- **Frontend:** React + Vite
- **Backend:** Express + Node.js 16
- **DB:** SQLite (`server/data/crm.db`)
- **Integração:** Evolution API (Docker containers separados — 3 containers + Postgres)
- **AI Agents:** OpenAI API pra mensagens automáticas

### Atenção especial

> ⚠️ **Evolution roda em Docker** (3 containers). Postgres do Evolution **crasha quando disco enche** (>80%). Sempre checar `df -h` antes de debugar "CRM não responde".

### Captura

`[INSERIR PRINT: tela inicial do CRM com lista de leads]`

---

## Sites de venda (estáticos)

Hospedados como HTML/JS puro (Apache + Cloudflare). Sem backend próprio — formulários postam pra:
1. **Google Sheets** (via Apps Script `/exec`)
2. **CRM Dros** (webhook `/crm/api/webhooks/sheets/<cliente>`)
3. **Meta CAPI** (Conversion API, se anúncio Meta)
4. **GTM** (Google Tag Manager, se configurado)

| Site | Domínio | Cliente |
|---|---|---|
| Box Paper | boxpaper.com.br | Box Paper Embalagens |
| Telhabras | telhabras.com.br | Telhabras Distribuidora |
| Gringa | gringacosmeticos.com.br | Gringa Cosméticos |
| Dros Sales | drosagencia.com.br/sales | Lead da própria agência |
| Lar do Sul | lardosul.com.br | Lar do Sul Imóveis |

> 💡 **Atualizar site estático:** sobe arquivos via FTP/cPanel. Não tem PM2 nem build.

---

## Outros sistemas menores na VPS

| Processo PM2 | O que é |
|---|---|
| `dros-oxi-pedidos` | Sistema interno de pedidos da Oxi Química |
| `gestao-clin` | Gestão clínica (cliente externo, projeto antigo) |

> 📌 Você raramente vai mexer nesses. Só lembrar que existem se algum cair e impactar disco/memória da VPS.

---

## Pré-requisitos pra acessar/operar

| Pra fazer isso | Você precisa de |
|---|---|
| Usar o Hub no navegador | Login dono/gerente/funcionário/cliente |
| Usar o /core | Login admin (`admin@drosagencia.com.br`) |
| Usar o CRM | Login admin ou de atendente |
| SSH na VPS | Chave SSH cadastrada + IP da VPS |
| Modificar código | Repositório clonado + Claude Code instalado |
| Deploy pra VPS | SSH + permissão de pull no GitHub |

---

## Próximo passo

Agora que você sabe o que existe, leia [03 — Como Usar Claude Code](03-COMO-USAR-CLAUDE-CODE.md) pra aprender o jeito Dros de modificar essas coisas.
