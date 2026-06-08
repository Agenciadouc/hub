# 07 — Funcionalidades do Hub

> Catálogo completo do que o Hub faz, organizado por seção do sidebar.

URL: https://drosagencia.com.br/hub

---

## Sidebar (navegação principal)

A sidebar agrupa funcionalidades em seções:

```
GESTÃO
├── Dashboard
├── Pipeline
├── Gravacoes (dono/gerente/funcionário)
├── Tarefas
├── Aprovacoes
└── Performance

ADMINISTRACAO (dono/gerente)
├── Clientes
├── Equipe
├── Departamentos
├── Categorias
├── Servicos
├── Financeiro (dono only)
├── Configuracoes (dono only)
└── CRM (link externo, dono only)
```

---

## Dashboard (`/hub/dashboard`)

**Quem acessa:** todos os roles.
**O que mostra (varia por role):**

### Pra dono/gerente
- **Sequência atual** — quantos dias seguidos com tarefa concluída (gamificação, ignora fim de semana)
- **Total de tarefas em risco** — overdue + ajustes pendentes
- **Tarefas atrasadas** — lista priorizada
- **Tarefas em risco hoje** — vence hoje
- **Heatmap de produção** — calor por dia da semana
- **Gráficos:**
  - Tarefas abertas por funcionário (com -30 hardcoded pra Graziele, que é gerente)
  - Concluídas no período por funcionário
  - Tempo médio em cada etapa
  - Taxa de retrabalho (% de tarefas que voltaram da aprovação pra revisão)

### Pra funcionário
- Sequência pessoal
- Tarefas atribuídas a ele
- Próximas gravações (se tiver)

### Pra cliente
- Tarefas aguardando aprovação dele
- Histórico

`[INSERIR PRINT: dashboard como dono]`

---

## Pipeline (`/hub/pipeline`)

**Quem acessa:** dono/gerente/funcionário.

**O que faz:** Kanban com as etapas configuradas em Configurações.

Etapas padrão:
1. Solicitação Pendente (cliente pediu, gerente precisa aprovar)
2. A fazer (backlog)
3. Em Produção
4. Ajustar (revisão interna)
5. Aprovação Interna (gerente revisa)
6. Aguardando Cliente
7. Aprovado pelo Cliente
8. Programar Publicação
9. Concluído (terminal)
10. Rejeitado (terminal)

### Tipos de tarefa
- **Normal** — tarefa única
- **Mãe** — agrupa subtarefas relacionadas
- **Linha Editorial** (workflow editorial automático) — mãe com 5 subtarefas iniciais + triggers dinâmicos
- **Recorrência** — botão que abre modal de template (vira tarefa periódica)

### Botões no header (dono/gerente/funcionário)
- ➕ **Nova Tarefa** — modal padrão
- 🧱 **Tarefa Mãe** — modal com subtarefas customizadas
- 🔁 **Recorrência** — modal de template (semanal/mensal)
- 📚 **Linha Editorial** (dono only) — modal hardcoded com 5 subs iniciais

### Drag & drop
Arrasta cartão entre colunas pra mudar etapa. Movimentos disparam:
- Notificações ao responsável
- Triggers de workflow editorial (se aplicável)
- Timer automático (entra em "Em Produção" → começa timer; sai → para)

`[INSERIR PRINT: pipeline com cartões em várias etapas]`

---

## Tarefas (`/hub/tasks`)

**Lista completa de tarefas com filtros.**

### Filtros disponíveis
- Cliente
- Departamento
- Etapa
- Responsável
- Categoria
- Prioridade
- Busca textual
- Data (de / até)

### Ações em massa
- Mover múltiplas pra outra etapa
- Atribuir múltiplas a um responsável

### Exportar
Botão **Exportar** gera Excel com todas as tarefas filtradas (dono only).

### Criar
Mesmos botões do pipeline (Nova Tarefa, Mãe, Recorrência).

`[INSERIR PRINT: tela Tarefas com lista e filtros]`

---

## Detalhe da Tarefa (`/hub/tasks/:id`)

Página de uma tarefa específica. Tabs:

### Detalhes
- Título, descrição (com quebras de linha preservadas)
- Cliente, departamento, categoria, prioridade
- Prazo, responsáveis
- Drive links (bruto e pronto)
- Conteúdo de aprovação (texto + arquivos do carrossel)
- Data e objetivo da publicação

### Comentários
- Internos (visível pra time)
- Cliente (visível pro cliente)

### Histórico
- Auditoria de mudanças de etapa
- Quem mudou, quando, comentário opcional

### Subtarefas (se for mãe)
- Lista com etapa atual
- Pode adicionar/editar/remover

### Tempo
- Timer manual (Iniciar / Pausar)
- Timer automático em "Em Produção"
- Histórico de sessões de trabalho

`[INSERIR PRINT: TaskDetail aberta com tabs]`

---

## Aprovações (`/hub/approvals`)

**Duas listas:**

### Aprovação Interna (dono/gerente)
Tarefas em etapa `aprovacao_interna`. Gerente revisa antes de enviar pro cliente.
- ✅ Aprovar → vai pra `aguardando_cliente`
- ❌ Rejeitar → volta pra `revisao_interna` (Ajustar)

### Aprovação Cliente (cliente)
Tarefas em etapa `aguardando_cliente`. Cliente vê o conteúdo + Drive embed.
- ✅ Aprovar → vai pra `aprovado_cliente`
- ✏️ Solicitar Alteração → cliente escreve o que mudar → volta pra `revisao_interna` com banner

### Aprovação pública (sem login)
Cliente pode aprovar sem login via token único: `drosagencia.com.br/approvals/<token>`.
Token gerado por dono/gerente em ClientDetail.

`[INSERIR PRINT: tela de aprovação cliente com carrossel]`

---

## Performance (`/hub/performance`)

**Painel de métricas das integrações.**

### Para dono/gerente
- Lista todos os clientes com pelo menos 1 ID vinculado
- Grid com cards: gasto Meta, sessões GA4, conversões GAds, seguidores IG
- Clica num cliente → detalhes daquela conta

### Para cliente
- Vê só a sua conta (filtrado por `core_*_id` da tabela `clients`)

### Tokens necessários
Configurados em Configurações → Tokens:
- `META_ACCESS_TOKEN` — Meta + IG
- `GOOGLE_ADS_*` (5 chaves) — GAds + GA4

> 💡 Tokens vencidos = painel mostra "Sem dados" ou erro. Renove em Configurações.

`[INSERIR PRINT: tela Performance com cards de clientes]`

---

## Gravações (`/hub/gravacoes`)

**Calendário de captação de vídeo.**

Mostra tarefas com `subtask_kind='gravacao'` OU departamento "Captação" + `recording_datetime` preenchido.

Vista de calendário (mês), arrastar pra reagendar.

`[INSERIR PRINT: calendário de gravações]`

---

## Clientes (`/hub/clients`)

**Cadastro completo de clientes.**

### Lista
- Filtro ativo/arquivado
- Busca
- Contagem de tarefas por cliente

### Detalhes (`/hub/clients/:id`)
Várias seções:

1. **Identificação** — nome, slug, logo
2. **Contato** — email, telefone, contato responsável
3. **Redes/Site** — Instagram, website
4. **Dados fiscais** — CNPJ, razão social, cidade/estado
5. **Contrato/Financeiro** — mensalidade, dia de pagamento, início do contrato
6. **Vínculos do Painel de Performance** — 4 dropdowns (Meta/IG/GAds/GA4) usando componente `CoreAccountSelect`
7. **Credenciais** — login/senha de plataformas do cliente (Meta Business, GA, etc), com máscara de senha (Eye/EyeOff)
8. **Serviços** — quais serviços a Dros presta pra esse cliente
9. **Usuários** — login do cliente no portal

### Ações
- Editar
- Arquivar (soft delete, `is_active=0`)
- Reativar
- Gerar/revogar token de aprovação pública
- Gerar token de onboarding

`[INSERIR PRINT: ClientDetail com vínculos do painel]`

---

## Equipe (`/hub/team`)

**Cadastro de funcionários e roles.**

### Roles
- `dono` — acesso total (CEO)
- `gerente` — acesso quase total, sem financeiro/configurações
- `funcionario` — sidebar limitada
- `cliente` — só portal de aprovações + dashboards limitados

### Departamentos
Cada funcionário pertence a 1+ departamentos. Departamentos têm cor e são usados pra:
- Atribuir tarefas
- Filtrar visualizações
- Configurar workflow editorial

`[INSERIR PRINT: tela Equipe com lista de usuários]`

---

## Departamentos, Categorias, Serviços

Taxonomia interna.

### Departamentos
Ex: DEV, CAPTAÇÃO, EDIÇÃO, ATENDIMENTO. Cor configurável.

### Categorias
Ex: NORMAL, LINHA EDITORIAL, RELATÓRIO MENSAL. Cor configurável.

### Serviços
Catálogo de serviços que a Dros oferece. Cada cliente assina N serviços com config própria.

---

## Financeiro (`/hub/financial`) — dono only

**DRE, despesas, receitas.**

- Receitas mensais por cliente (vem do `monthly_fee`)
- Despesas fixas (cadastra recorrente)
- Despesas variáveis (avulsa)
- Receitas extras
- Parcelamentos
- Dashboard com gráficos

Todas operações têm `paid_at` (data de pagamento) — pode marcar como pendente ou pago.

`[INSERIR PRINT: dashboard financeiro]`

---

## Configurações (`/hub/settings`) — dono only

**3 abas:**

### 1. Etapas do Pipeline (default)
- Lista de etapas com cor + slug
- Drag pra reordenar
- Marca etapa como "terminal" (concluído, rejeitado)
- Botão Editar → modo edição (adicionar/remover/renomear)

### 2. Tokens / Integrações
- **Meta / Instagram:** Meta Access Token
- **Google Ads + GA4:** Developer Token, Client ID, Client Secret, Refresh Token, Login Customer ID
- Cada campo:
  - Mascarado (`****ktp`) por padrão
  - Botão Eye/EyeOff pra mostrar quando editando
  - Status: ✓ Configurado / Fallback .env / Não configurado
  - Acordeão "Como obter" com passo a passo
- Salva sem precisar restart (lê do DB a cada call)

### 3. Contas do Painel
- Lista clientes do painel (mesmos da tabela `clients`)
- Badges Meta/IG/GAds/GA4 por linha (colorido = vinculado)
- Search por nome, slug ou ID
- Toggle "Mostrar arquivadas"
- **+ Nova Conta** — modal com nome, slug, email + 4 `CoreAccountSelect` (search direto da Meta/Google API)
- **Editar** — mesma modal pré-preenchida
- **Arquivar** — soft delete

`[INSERIR PRINT: aba Tokens]`
`[INSERIR PRINT: aba Contas do Painel com modal aberto]`

---

## Tarefas Recorrentes

Não é uma página própria — é uma **funcionalidade transversal** acessada via botão **Recorrência** no Pipeline e em Tarefas.

### Como funciona
1. Clica em **Recorrência** → modal abre
2. Define:
   - Nome do template (interno)
   - Tipo: Normal ou Mãe (com subtarefas pré-configuradas)
   - Frequência: Semanal ou Mensal
   - Dia (1-7 semanal, 1-31 mensal)
   - Hora (default 06:00)
   - Prazo (+N dias após criação)
   - Cliente, depto, responsáveis, etc
3. Salva → template fica em `task_templates`

### Cron
A cada 5 min, o Hub verifica templates com `next_run_at <= now` e cria a tarefa real.

### Página de gerenciamento
`/hub/tarefas-recorrentes` (não tem mais no sidebar, mas a rota existe) — lista todos os templates, executar agora, editar, arquivar.

`[INSERIR PRINT: modal Nova Recorrência]`

---

## Notificações

Sino na sidebar com contagem em tempo real (SSE).

### O que notifica
- Tarefa atribuída a você
- Tarefa precisa de aprovação (pra gerente)
- Cliente solicitou alteração
- Tarefa virou overdue
- Timer rodando > 1h (pergunta "ainda em produção?")

`[INSERIR PRINT: dropdown de notificações]`

---

## CRM (link externo)

Atalho no sidebar (dono only) abre `/crm/` em nova aba. Sistema separado, ver [09 — CRM](09-FUNCIONALIDADES-CRM.md).

---

## Workflow Editorial (hardcoded)

Sistema especial pra clientes de social media.

Tarefa mãe `task_type='mae_editorial'` cria 5 subtarefas iniciais:
1. **Briefing** → Ivandro
2. **Reunião Aprovação Cliente** (do briefing) → Ivandro
3. **Aprovação Interna Final** → Ivandro
4. **Aprovação Cliente (Final)**
5. **Publicação** → Graziele

### Triggers dinâmicos (via PUT /tasks/:id/stage)
- Briefing → Criar Imagens (Dalila) paralelo
- Criar Imagens → Programar Publ Imagens (Graziele)
- Reunião → Gravação (Ivandro)
- Gravação → Subir Arquivos
- Subir Arquivos → Editar Vídeos
- Editar Vídeos → Programar Publ Vídeos (Graziele)

Quando todas as 11 subtarefas conhecidas concluem, mãe auto-fecha.

> 📌 Lista de nomes hardcoded por `LIKE`: Ivandro, Dalila, Graziele. Se trocar funcionário, precisa atualizar no código.

---

## APIs internas do Hub

Lista das rotas principais (pra quem precisa integrar):

| Rota | Método | O que faz |
|---|---|---|
| `/api/auth/login` | POST | Login (retorna JWT) |
| `/api/tasks` | GET | Lista tarefas com filtros |
| `/api/tasks/:id` | GET | Detalhe tarefa |
| `/api/tasks/:id/stage` | PUT | Move etapa |
| `/api/clients` | GET/POST | CRUD clientes |
| `/api/settings` | GET/PUT | Tokens (dono only) |
| `/api/config/tokens` | GET | Tokens em claro (com X-Core-Secret) |
| `/api/performance/meta/accounts` | GET | Lista contas Meta |
| `/api/task-templates` | GET/POST/PUT/DELETE | Recorrências |
| `/api/approvals/internal` | GET | Aprovações pendentes |
| `/api/notifications` | GET | Lista notificações |
| `/api/sse` | GET | Server-sent events (notificações) |

Todas (menos `/login` e `/approvals/:token`) precisam de JWT no header `Authorization: Bearer <token>`.

---

## Próximo passo

[08 — Funcionalidades do /core](08-FUNCIONALIDADES-CORE.md).
