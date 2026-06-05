# 11 — Glossário

> Dicionário de termos técnicos em linguagem leiga. Use como consulta sempre que tropeçar num termo.

---

## A

### API
Sigla pra "Application Programming Interface". É como dois sistemas conversam entre si. Quando o Hub busca dados do Meta Ads, ele usa a API do Facebook. Quando você usa a API do Hub, está chamando um endpoint tipo `/api/tasks`.

**No nosso contexto:** Meta Graph API, Google Ads API, GA4 API, API interna do Hub/Core/CRM.

### Apps Script
Linguagem de programação do Google pra automatizar planilhas/docs. Cada site da Dros tem um Apps Script que recebe leads do form e envia pro CRM.

---

## B

### Backend
A parte do sistema que **roda no servidor** (não no navegador). Processa requisições, acessa o banco de dados, chama APIs externas. No nosso caso: Express + Node.js.

### Branch
"Galho" do código. Linha do tempo paralela. A Dros usa só uma branch: `master`.

### Build
Processo que transforma o código de desenvolvimento (`.tsx`, `.ts`) em arquivos otimizados (`.js`, `.css`) que o navegador entende. Pra Hub e /core, build acontece **local** antes do commit. Pra CRM, na VPS.

### Bundle
Arquivo final gerado pelo build. Geralmente um `.js` gigante com tudo do frontend junto. Ex: `agency-hub/dist/assets/index-abc123.js`.

---

## C

### Cache
Cópia temporária pra acelerar acessos repetidos. Navegador faz cache de imagens, Hub faz cache de tokens, /core faz cache de respostas de API.

### CAPI (Conversion API)
API de Conversão do Meta. Os sites de venda enviam eventos de conversão direto pro Meta (em vez de só via pixel no navegador), pra melhorar tracking quando pixel é bloqueado.

### Claude Code
A IA da Anthropic que rodamos no terminal. Lê código, conversa em português, propõe planos e implementa. Ver [03 — Como Usar Claude Code](03-COMO-USAR-CLAUDE-CODE.md).

### Cloudflare
Camada de CDN/proxy que fica na frente do nosso domínio. Acelera resposta global e protege contra ataques.

### Commit
Um "salvamento" do código no Git. Tem autor, data, mensagem descritiva e o conjunto de mudanças. Exemplo: `feat(agency-hub): adiciona aba Tokens`.

### Cron
Sistema que executa tarefas em horários programados. No Hub, cron implementado via `setInterval` JavaScript roda a cada 5min pra criar tarefas recorrentes.

### CRUD
Sigla de Create, Read, Update, Delete. As 4 operações básicas de qualquer cadastro. Hub tem CRUD de clientes, tarefas, etc.

---

## D

### DB / Banco de Dados
Onde os dados ficam armazenados de forma estruturada. A Dros usa **SQLite** (arquivo único `.db`) por simplicidade. Cada sistema tem seu DB:
- Hub: `agency-hub/server/data/hub.db`
- CRM: `crm-dashboard/server/data/crm.db`
- /core: nenhum (lê de APIs externas)

### Deploy
Subir código pra produção. No nosso caso: `git push` + `git pull` na VPS + `pm2 stop+start`.

### Diff
Mostra **o que mudou** entre duas versões do código. `git diff` mostra mudanças antes do commit, GitHub mostra diff de pull requests, Claude mostra diff dos arquivos modificados.

### Docker
Sistema de containers. Roda apps isolados em "caixas" leves. Evolution API (do CRM) roda em 3 containers Docker.

### dotenv (.env)
Arquivo com variáveis de ambiente em formato `CHAVE=valor`. Não vai pro git. Cada sistema tem o seu:
- Hub: `/opt/platform/agency-hub/.env`
- /core: `/root/core/.env` (não `client-dashboard/.env`!)
- CRM: `/root/crm/crm-dashboard/.env`

---

## E

### Endpoint
Uma URL específica de uma API que faz algo. Ex: `GET /api/tasks` é o endpoint que lista tarefas.

### Evolution API
Gateway WhatsApp não-oficial que roda em Docker. CRM da Dros usa ele pra mandar/receber mensagens.

### Express
Framework Node.js pra criar servidores HTTP. Hub e /core usam Express 5, CRM usa Express 4.

---

## F

### Fallback
"Plano B" automático. Se sistema novo falha, usa o velho. Ex: se DB de tokens vazio, lê do `.env`.

### Frontend
A parte do sistema que **roda no navegador**. HTML, CSS, JavaScript que o usuário vê. No nosso caso: React.

---

## G

### GA4 (Google Analytics 4)
Versão atual do Google Analytics. Hub busca métricas via API com OAuth Refresh Token compartilhado com Google Ads.

### Git
Sistema de versionamento. Salva todas as versões do código com histórico completo. Ver [05 — Git e Deploy](05-GIT-E-DEPLOY.md).

### GitHub
Onde nosso código mora online. Três repos: `platform`, `core`, `crm`.

### GTM (Google Tag Manager)
Ferramenta pra gerenciar tags de analytics/conversão sem mexer no código do site. Alguns sites de venda usam.

---

## H

### Hash
Texto curto que identifica unicamente um commit. Ex: `c2bd991`. Use pra referenciar versões específicas.

---

## I

### IDE
Editor de código com superpoderes. VS Code é a IDE recomendada na Dros.

### iframe
Elemento HTML que embute uma página dentro de outra. Hub usa pra mostrar dashboards do /core dentro de páginas do Hub.

---

## J

### JSON
Formato de dados em texto. Quase tudo na web é JSON. Ex:
```json
{ "nome": "João", "id": 42 }
```

### JWT (JSON Web Token)
Token de autenticação assinado. Quando você faz login, o servidor te dá um JWT. Você manda esse token em toda requisição seguinte pra provar quem você é. Hub e /core usam JWT.

---

## L

### Lead
Pessoa que demonstrou interesse mas ainda não comprou. CRM gerencia leads.

### Log
Texto que o sistema escreve dizendo o que tá fazendo. `pm2 logs <processo>` mostra logs de cada sistema.

---

## M

### Meta Graph API
API oficial do Meta (Facebook + Instagram). Hub usa pra puxar métricas de ads, posts, seguidores etc.

### Migration
Script que muda a estrutura do banco de dados (adiciona tabela, coluna, etc). Nossas migrations rodam automaticamente no startup do servidor e são **idempotentes** (rodar de novo não quebra).

### MCC (Google Ads)
"My Client Center" — conta gerenciadora do Google Ads. Permite acessar contas de vários clientes com um único login.

---

## N

### Node.js
Ambiente pra rodar JavaScript no servidor. Hub, /core e CRM são Node.js 16.x.

### npm
Gerenciador de pacotes do Node. `npm install` baixa dependências. `npm run build` roda script de build.

---

## O

### OAuth
Padrão de autorização entre serviços. Quando você "Logar com Google" em algum site, é OAuth. Hub usa OAuth pra acessar Google Ads e GA4.

### Onboarding
Processo de cadastrar cliente novo. Hub tem token de onboard que gera link único pra cliente preencher dados próprios.

---

## P

### Plan Mode
Modo do Claude Code onde ele só investiga e propõe sem fazer mudanças. Você aprova depois. Ver [03 — Claude Code](03-COMO-USAR-CLAUDE-CODE.md).

### PM2
Gerenciador de processos do Node.js na VPS. Mantém Hub/Core/CRM rodando 24/7, reinicia se cair, gerencia logs. Comandos: `pm2 status`, `pm2 logs`, `pm2 stop`, `pm2 start`, `pm2 flush`.

### Production / Produção
O sistema rodando na VPS, sendo usado por clientes/funcionários reais. **Nunca testar em produção sem backup.**

### Pull
Trazer commits do GitHub pra local (ou pra VPS).

### Push
Mandar commits locais pro GitHub.

---

## R

### Refresh Token
Token de longa duração que permite gerar Access Tokens curtos sem precisar logar de novo. Google Ads e GA4 usam.

### Remote
Servidor onde mora o repositório. Cada repo nosso tem um remote: `platform`, `core`, `crm`.

### Repo / Repository
O projeto inteiro no Git (código + histórico). Cada sistema é um repo.

### Restart vs Stop+Start
**Restart:** PM2 mantém o processo "ligado" durante a operação, às vezes cache env velho.
**Stop+Start:** mata e sobe de novo, mais lento mas mais confiável. **Prefere stop+start.**

### Rollback
Voltar atrás uma mudança que deu errado. `git revert HEAD` + push é rollback.

### Route / Rota
URL que o servidor responde. Ex: `GET /api/tasks` é uma rota.

---

## S

### Schema
Estrutura de uma tabela no DB. Quais colunas, tipos, restrições.

### Seed
Popular o DB com dados iniciais. Quando Hub liga, ele faz seed dos tokens do `.env` pra tabela `app_settings` se ainda não existem.

### Slug
Versão "URL-amigável" de um texto. Ex: nome "Box Paper Embalagens" → slug "box-paper-embalagens".

### Soft Delete
"Apagar" sem remover do DB. A gente marca `is_active=0`. Permite recuperar depois.

### SQL
Linguagem pra consultar bancos relacionais. SQLite, Postgres, MySQL usam SQL.

### SQLite
DB em arquivo único. Não precisa rodar servidor separado. Hub e CRM usam SQLite.

### SSE (Server-Sent Events)
Tecnologia pro servidor empurrar mensagens pro navegador em tempo real (sem o navegador ficar perguntando). Hub usa pra notificações.

### SSH
Forma de conectar remotamente a um servidor via terminal. `ssh root@vps-...`. Precisa de chave ou senha.

### Stack
Conjunto de tecnologias usadas. Stack do Hub: React + Express + SQLite + Node.

### Subagent
Mini-Claude que roda uma tarefa específica. Plan Mode usa subagent `Explore` pra investigar código sem poluir contexto principal.

---

## T

### Tag (HTML/Git)
- **HTML:** elemento como `<div>`, `<button>`.
- **Git:** marca de versão importante (raramente usado na Dros).

### Token
Texto secreto que prova quem você é ou autoriza acesso. Tipos:
- **JWT:** sessão de login (Hub/Core/CRM)
- **Access Token:** acesso a API externa (Meta, Google)
- **Refresh Token:** regenera Access Token
- **Embed Token:** auto-login no /core via iframe

### TypeScript
JavaScript com tipos. Detecta erros antes de rodar. Frontend do Hub é TypeScript.

---

## V

### VPS (Virtual Private Server)
Servidor remoto onde tudo da Dros roda. HostGator é nosso provedor. IP: vps-5269157.3store.com.br.

---

## W

### Webhook
URL que recebe notificações de outro sistema. CRM tem webhook que recebe leads de sites: `POST /crm/api/webhooks/sheets/<cliente>`.

---

## Próximo passo

[12 — Receitas Passo a Passo](12-RECEITAS-PASSO-A-PASSO.md) — cookbook operacional.
