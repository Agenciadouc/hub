# 09 — Funcionalidades do CRM

> Gerenciador de leads do WhatsApp via Evolution API.

URL: https://drosagencia.com.br/crm
Repo: `crm-dashboard/`
Processo PM2: `dros-crm` (porta 3002)

---

## Pra que serve

O CRM da Dros gerencia **leads vindos do WhatsApp**. É a ponte entre:

```
Site de venda → Form submit → Apps Script
       ↓                        ↓
   Google Sheets ← (a cada 5min) ↓
                                 ↓
                          CRM Dros recebe webhook
                                 ↓
                          Atribui atendente (humano ou IA)
                                 ↓
                          Conversa via Evolution API
                                 ↓
                          Lead vira venda (ou não)
```

Quem usa: equipe comercial / SDR / atendimento.

---

## Login

URLs e usuários cadastrados em produção. Veja com o gerente comercial.

Roles:
- **admin** — vê tudo, configura atendentes
- **atendente** — vê só seus leads atribuídos

---

## Páginas principais

### Dashboard (`/crm`)
Métricas gerais:
- Leads recebidos hoje / semana / mês
- Leads atribuídos vs não atribuídos
- Taxa de conversão por atendente
- Tempo médio de primeira resposta

### Chat (`/crm/chat`)
Interface tipo WhatsApp Web:
- Sidebar com lista de leads (filtros: meus, todos, por status)
- Painel central: histórico da conversa
- Painel direito: dados do lead (nome, fonte, tags, observações)

`[INSERIR PRINT: tela Chat do CRM]`

### Integrations (`/crm/integrations`)
Configurações:
- Conexão Evolution API (status do container, número conectado)
- Webhooks de sites (URL do CRM, regras de atribuição)
- Templates de mensagem automática

### Admin (admin only)
- Global Dashboard — métricas de todas as contas
- Cadastro de atendentes (humanos e IA)
- Cadastro de clientes (mapeamento webhook → account)
- Logs de erros

---

## Atendentes

Existem dois tipos:

### Atendente humano
- Login próprio
- Recebe leads via atribuição (round-robin ou regra de horário)
- Conversa pelo CRM, mensagens vão pro Evolution
- Métricas: leads atribuídos, taxa de resposta, conversão

### Atendente IA (agent)
- Configurado por admin: prompt, modelo (OpenAI gpt-4 etc), regras
- Responde automaticamente quando lead chega fora do horário
- Pode escalar pra humano via comando (tipo `/humano`)
- Métricas separadas

`[INSERIR PRINT: tela de cadastro de agente IA]`

---

## Follow-ups automáticos

Se lead não responde por X horas, CRM envia mensagem automática.

### Como configurar
1. Admin → Integrations → Follow-ups
2. Define template + delay (ex: 4h, 24h, 72h)
3. Pode ter sequência (1ª msg, 2ª, 3ª, parar)

### Como funciona
1. Scheduler verifica leads inativos a cada 10min
2. Manda template via Evolution
3. Marca como "follow-up enviado" pra não repetir
4. Se lead responde, scheduler para

---

## Integração com sites de venda

Cada site tem um **Apps Script** rodando na planilha do cliente:

### Fluxo
1. Visitante preenche form no site
2. Form posta JSON na URL `/exec` do Apps Script
3. Apps Script salva linha na Google Sheet
4. Trigger 5min: Apps Script envia leads novos pro CRM:
```
POST https://drosagencia.com.br/crm/api/webhooks/sheets/<cliente>
{
  "nome": "...",
  "telefone": "5548...",
  "fbclid": "...",
  ...
}
```
5. CRM recebe, cria lead, atribui atendente

### Sites integrados (parcial)
| Site | Cliente | Slug webhook |
|---|---|---|
| boxpaper.com.br | Box Paper | `box-paper` |
| telhabras.com.br | Telhabras | `telhabras` |
| gringacosmeticos.com.br | Gringa | `gringa` |
| drosagencia.com.br/sales | Dros (auto-venda) | `dros-sales` |

Slug é o **"Nome textual (apelido)"** do cliente no Hub. Tem que bater exatamente.

---

## Evolution API (gateway WhatsApp)

### Containers (Docker)
3 containers rodando na mesma VPS:
- `evolution-api` — server principal
- `evolution-redis` — cache de sessões
- `evolution-postgres` — DB de mensagens

### Atenção crítica
> ⚠️ **Postgres do Evolution crasha quando disco enche.** Sempre `df -h` antes de debugar problemas de CRM. Se passou 85%, prioridade é limpar disco.

### Comandos úteis
```bash
# Status dos containers
docker ps | grep evolution

# Logs do Evolution
docker logs evolution-api --tail 50

# Reiniciar
docker restart evolution-api
```

---

## Stack técnica

| Aspecto | Detalhe |
|---|---|
| Node | 16.x via nvm |
| Frontend | React + Vite |
| Backend | Express + Node.js |
| DB | SQLite (`server/data/crm.db`) |
| WhatsApp | Evolution API (Docker) |
| AI Agents | OpenAI API (gpt-4o-mini default) |
| Auth | JWT |

---

## Endpoints principais

| Endpoint | O que faz |
|---|---|
| `POST /api/webhooks/sheets/:cliente` | Recebe leads dos sites (via Apps Script) |
| `POST /api/webhooks/evolution` | Recebe mensagens do WhatsApp (Evolution) |
| `GET /api/leads` | Lista leads do atendente |
| `POST /api/leads/:id/messages` | Envia mensagem (CRM → Evolution → WhatsApp) |
| `GET /api/agents` | Lista agentes IA |
| `POST /api/follow-ups` | Configura follow-up |
| `GET /api/metrics/attendant/:id` | Métricas de atendente |

---

## Deploy do CRM

> ⚠️ Diferente do Hub: **build acontece NA VPS**, não local.

```bash
# Local — só commit + push (sem build)
git add crm-dashboard/
git commit -m "feat(crm): ..."
git push crm master

# VPS
ssh root@vps-...
cd /root/crm
git pull origin master
cd crm-dashboard
npm run build
cd /root/crm
pm2 stop dros-crm && pm2 start dros-crm
```

### Por que diferente do Hub
Histórico — CRM começou assim. Custa um pouco mais de CPU na VPS durante o build, mas funciona. Migrar pra padrão Hub é roadmap futuro.

---

## Quando você vai mexer

- Adicionar novo cliente (mapear webhook): admin → cadastrar cliente
- Configurar novo agente IA: admin → cadastrar agente
- Ajustar template de follow-up
- Debugar lead que não chegou (Apps Script trigger? Evolution? webhook?)

---

## Quando NÃO mexer

- Não fuçar nos containers Evolution sem entender (risco de quebrar WhatsApp)
- Não dropar tabelas sem backup
- Não mudar slug de cliente sem confirmar que Apps Script bate

---

## Próximo passo

[10 — Troubleshooting](10-TROUBLESHOOTING.md) — quando alguma coisa quebrar.
