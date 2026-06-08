# Histórico do Ecossistema Dros

> Linha do tempo das implementações importantes. **Todas conduzidas por João Luiz Soares de Mattos** com auxílio do Claude Code.

---

## 2025 (pré-2026)

### Construção dos sistemas iniciais
- **Hub** (`platform`) — sistema principal da agência
- **/core** (`core`) — Painel de Performance standalone
- **CRM** (`crm`) — gerenciamento de leads WhatsApp

Stack escolhida pra todos:
- Node.js 16 na VPS HostGator
- SQLite como DB (simplicidade > escala)
- React + Vite no frontend
- PM2 pra processos 24/7
- JWT pra auth

---

## Abril 2026

### Migração Performance: /core → Hub
**Motivação:** /core era painel isolado. Cliente que ia no Hub precisava abrir outro navegador pra ver métricas. Decidimos integrar nativamente.

**Implementação:**
- Páginas `PerformanceOverview.tsx` e `Performance.tsx` no Hub
- Rotas `/api/performance/*` (Meta, IG, GAds, GA4)
- Filtragem por role: admin vê tudo, cliente vê só sua conta (via `core_*_id` na `clients` table)
- `/core` mantido em paralelo (mudanças espelhadas em ambos)

**Memória:** `project_performance_migration.md` — política de espelhar mudanças.

### Vínculos por ID
**Motivação:** matcher por nome era frágil (mudança de nome no Meta quebrava). Migrado pra IDs explícitos.

**Implementação:**
- Tabela `clients` ganhou `core_meta_account_id`, `core_ig_page_id`, `core_gads_customer_id`, `core_ga4_property_id`
- Componente `CoreAccountSelect` com search direto na API

---

## Maio 2026

### Carrossel de aprovação (multi-files)
**Motivação:** post de carrossel no IG tinha vários assets — antes só dava pra mandar 1 link de aprovação.

**Implementação:**
- Campo `approval_files` (JSON array) na tabela `tasks`
- UI com checkbox "É carrossel?" + N inputs de link
- Renderização swipe horizontal no portal de aprovação

### Mãe genérica com subtarefas customizadas
**Motivação:** workflow editorial era rígido (5 subs hardcoded). Precisava de mãe pra projetos custom.

**Implementação:**
- `task_type='mae'` (além de `mae_editorial` existente)
- Subtarefas configuráveis no modal de criar mãe
- Não dispara triggers automáticos (diferente do editorial)

### Retrabalho no Dashboard
**Motivação:** queremos métrica de "quantas tarefas voltaram da aprovação pra revisão" pra identificar gargalos.

**Implementação:**
- Query usando `task_history` (transições de etapa)
- Card no Dashboard mostrando %
- Lista de tarefas com retrabalho recente

### Fix: subtasks em pipeline ativo
**Bug:** Pipeline só mostrava primeira subtask por `subtask_position`. Se sub2 estava em `revisao_interna` e sub1 em `concluido`, sub2 não aparecia.

**Fix:** query expandida pra mostrar todas as subs em etapas ativas (não só primeira não-concluída).

### Bug fixes diversos
- Modal não fechava quando arrastava mouse pra apagar texto
- Streak ignorando fim de semana (sábado/domingo)
- Financeiro respeitando `contrato_inicio` (não mostrar cliente antes do contrato)
- Timer-check rodando a cada 4s (na verdade era 7200s, mas guard fazia disparar 0-60s)

### Tarefas para apagar (IDs 467, 656, 570, 620, 621)
Tarefas de teste apagadas via SQL com [Receita 36](12-RECEITAS-PASSO-A-PASSO.md#receita-36).

---

## Maio/Junho 2026 — Sprint Tarefas Recorrentes

### Sistema de Tarefas Recorrentes
**Motivação:** linha editorial mensal, relatórios semanais, lembretes recorrentes — tudo recriado do zero todo mês/semana. Desperdício de tempo + fonte de inconsistência.

**Solução:** templates de recorrência que geram tarefas automaticamente.

**Implementação (fases):**
1. **Backend core:**
   - Tabelas `task_templates`, `task_template_assignees`, `task_template_subtasks`, `task_template_subtask_assignees`
   - Função `computeNextRunAt(type, day, hour)` — calcula próxima execução em BRT, com fallback pra último dia do mês se dia=31 em fev
   - Função `createTaskFromTemplate(templateId)` — clona template em tarefa real
   - `runDueTemplates()` chamada pelo cron
2. **Rotas CRUD:** `routes/task-templates.js` com GET/POST/PUT/DELETE + `/run-now`
3. **Cron scheduler:** `setInterval` a cada 5min no `server/index.js`
4. **Frontend:**
   - Página `TaskTemplates.tsx` (lista + modal)
   - Componente `TaskTemplateModal.tsx` extraído pra reuso
   - Botão "Recorrência" no Pipeline e Tarefas
5. **Memória de bug:** schema parcial detectado, drop+recria automático

**Commits chave:**
- `99b4648` — primeira versão
- `9b69579` — detecta schema parcial e recria
- `c2bd991` — botão Recorrência no Pipeline/Tasks, libera funcionário
- `5878bf7` — fix partial index

**Plano arquivado:** `archive-tarefas-recorrentes.md`.

---

## Junho 2026 (semana 1)

### 4 clientes novos no Painel
**Motivação:** Box Paper Embalagens, Ambiental Higiene, Gringa Cosméticos, Telhabras assinaram contrato e precisavam aparecer no Painel.

**Implementação:**
- Meta IDs descobertos via Meta MCP (Anthropic Connector)
- IG IDs e GA4 IDs fornecidos pelo João Luiz
- Cadastro via SQL direto no DB (mais rápido que UI)
- Substrings adicionadas no `ALLOWED_CLIENTS` do /core

**Commit:** `f943f61` — adiciona ao ALLOWED_CLIENTS.

### Fix Express 5 wildcard
**Bug:** /core caiu após upgrade Express 5 — `app.get('/core/*', ...)` deixou de funcionar.

**Fix:** mudança pra `app.get('/core/{*path}', ...)`.

**Lição aprendida:** sintaxe path-to-regexp mudou. [Receita 43](12-RECEITAS-PASSO-A-PASSO.md#receita-43).

### Renovação Token Meta (60 dias)
**Trigger:** token expirou. Performance caiu.

**Processo:**
1. Graph API Explorer pra gerar curto
2. Curl exchange pra long-lived
3. Cola no `.env` (UI ainda não existia)

**Bug colateral:** `==` duplo no `.env` (typo no nano). Token "Cannot parse access token". Fix via SQL — [Receita 41](12-RECEITAS-PASSO-A-PASSO.md#receita-41).

---

## Junho 2026 (semana 2) — Sprint Configurações

### Aba Configurações no Hub
**Motivação:** trocar token virou drama de SSH+nano. Cadastrar cliente exigia editar código (ALLOWED_CLIENTS). Sem proteção visual de tokens.

**Solução:** UI nova em Configurações com gerenciamento completo.

**Implementação (3 fases):**

**Fase 1 — Backend de tokens (MVP)**
- Tabela `app_settings` (key, value, is_secret, updated_at)
- `routes/settings.js` com `KNOWN_KEYS` (Meta + 5 GAds, Kiwify removido depois)
- `GET /api/settings` retorna mascarado (`****ktp`)
- `PUT /api/settings/:key` atualiza (dono only)
- `performance.js` ajustado: `getMetaToken()`/`getGADS()` lê DB com fallback `.env`
- Frontend: `Settings.tsx` virou tabs (Pipeline + Tokens)
- Componente `TokensTab.tsx` com Eye/EyeOff + acordeão "Como obter"

**Fase 2 — Cadastro de contas**
- Aba "Contas do Painel"
- Componente `AccountsTab.tsx`
- Reaproveita `CoreAccountSelect` (search direto da Meta/Google API)
- CRUD: criar, editar, arquivar (soft delete)

**Fase 3 — /core dinâmico**
- Endpoint `/api/config/tokens` e `/api/config/clients` no Hub (autenticado via `X-Core-Secret`)
- /core: função `syncFromHub()` chamada no startup + a cada 10min
- Substitui `META_TOKEN` hardcoded por `let` que sync atualiza
- `GADS` virou objeto com `get`-getters pra refletir mudanças runtime
- `ALLOWED_CLIENTS` hardcoded + `HUB_CLIENT_NAMES` dinâmico (OR)

**Bonus:** auto-seed do `.env` pro DB no startup. Função `seedSettingsFromEnv()` em `index.js`. Idempotente.

**Reordenação tabs:** Etapas do Pipeline = primeira, padrão de abertura.

**Commits chave:**
- `16f37f7` — aba Tokens
- `b91df75` — aba Contas + endpoint /api/config
- `0df1203` — auto-seed do .env
- `2808fce` — reordena tabs

### -30 nos gráficos da Graziele
**Motivação ad-hoc:** gráfico "Tarefas Concluídas por Funcionário" mostra Graziele como top, mas ela é gerente (concluiu por papel de aprovação interna, não produção). Visualmente injusto pros funcionários operacionais.

**Implementação:** hardcoded -30 na barra dela (match por nome `includes('grazi')`). Marcado como "chore" (não é feature pública).

**Commit:** `a928b04`.

### White-space pre-wrap em descrições
**Bug:** quebras de linha na descrição/comentários da tarefa eram colapsadas (CSS `white-space: normal`).

**Fix:** `style={{ whiteSpace: 'pre-wrap' }}` em descrição da tarefa + comentários internos + comentários cliente.

**Commit:** `aa23298`.

---

## Junho 2026 (esta semana — sprint Documentação)

### Documentação completa
**Motivação:** só o João Luiz sabe como tudo funciona. Risco operacional. Funcionário novo não tem como começar.

**Implementação:**
- Pasta `docs/` com 14 arquivos markdown
- Cobertura: setup, workflow, deploy, operação, funcionalidades, troubleshooting, glossário, receitas
- Linguagem leiga, exemplos copiáveis, capturas marcadas
- Cookbook com 44+ receitas reproduzíveis

**Tu tá lendo a doc agora 👋.**

---

## Memórias importantes do Claude

Memórias persistentes que orientam todo trabalho:

### Feedback
- `feedback_grep_all_occurrences.md` — sempre grep todas ocorrências antes de fix
- `feedback_no_claude_coauthor.md` — NUNCA `Co-Authored-By: Claude` em commits (autoria limpa)
- `feedback_ship_everything.md` — em pedidos de deploy, subir tudo do working tree
- `feedback_pm2_restart_caching.md` — `pm2 restart` não confiável, usar stop+start

### Project state
- `project_dros_launch.md` — funil Nayara (Green) + Josi (Kiwify)
- `project_josi_tracking_progress.md` — GTM + CAPI + aulão Josi
- `project_nayara_assets.md` — assets Nayara (Kiwify, YT, WhatsApp)
- `project_bot_sdr_plano.md` — Bot SDR plano futuro CRM
- `project_performance_migration.md` — Performance migrado /core→/hub paralelo
- `project_emdi_tracking.md` — EMDI GA4 consertado de G-HP8DNNVL9F → G-WJ5RQXBRZE

### Reference
- `reference_vps_infra.md` — 3 sistemas (Hub:3003, CRM:3002, Core:3004), Apache, Node 16
- `reference_hub_deploy.md` — fluxo deploy Hub
- `reference_core_deploy.md` — fluxo deploy /core
- `reference_crm_deploy.md` — fluxo deploy CRM
- `reference_evolution_infra.md` — Evolution Docker (3 containers), Postgres crash = disco cheio
- `reference_freepik.md` — login Freepik pra imagens
- `reference_hub_landing.md` — landing de vendas em `agency-hub/landing.html`

---

## Padrões que viraram cultura

1. **Spec-driven** — Plan Mode antes de implementar
2. **Backup before destructive** — sempre `cp hub.db hub.db.backup-...` antes de SQL destrutivo
3. **Stop+start, não restart** — PM2 cache de env é traiçoeiro
4. **Fallback first** — DB com fallback `.env`, hardcoded com fallback API
5. **Migrations idempotentes** — `try { db.exec(...) } catch {}`
6. **Build local pra Hub e /core** — `dist/` vai commitado
7. **Commits com prefixo** — `feat(escopo):` / `fix(escopo):` / `chore(escopo):` / `docs:`
8. **Doc no mesmo commit** — feature importante atualiza `docs/`
9. **Memória do Claude** — fatos relevantes salvos pra contextos futuros
10. **Atribuição limpa** — sem `Co-Authored-By: Claude`, autoria é do humano

---

## Roadmap (provável)

Itens já planejados mas não implementados (ordem de prioridade):

- [ ] Bot SDR no CRM (piloto Oxi Química) — plano completo em `project_bot_sdr_plano.md`
- [ ] CRM build local (alinhar com padrão Hub)
- [ ] Histórico de tarefas geradas por template (link "ver tarefas geradas")
- [ ] Encriptação real dos tokens no DB (hoje só máscara visual, valor cleartext)
- [ ] Multi-user no /core (hoje só `admin`)
- [ ] Recorrência por intervalo arbitrário ("a cada 3 dias")
- [ ] Export PDF dos dashboards
- [ ] Auto-renovação de token Meta (System User como default + alertas de vencimento)

---

## Como atualizar esse histórico

Toda feature importante (que mexe em DB, adiciona página, muda comportamento) ganha entrada aqui no MESMO commit da implementação.

Padrão:
```markdown
### <Título da feature>
**Motivação:** [por quê]
**Implementação:** [resumo do como, com commit hash]
**Aprendizados:** [opcional — bugs encontrados, decisões tomadas]
```

> 💡 Esse histórico é o "porquê" de tudo. Daqui a 1 ano, alguém vai abrir e entender. Vale o esforço de manter.
