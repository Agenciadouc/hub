# 04 — Nosso Workflow de Desenvolvimento

> O jeito Dros de construir e manter software. **Spec-driven + IA + backup + rollback.**

---

## Filosofia em 1 parágrafo

Não acreditamos em "vamos codar e ver no que dá". Toda mudança técnica começa com **um plano escrito**, segue por **fases incrementais com backup antes**, é **testada local e em produção**, e tem **rollback pronto** caso quebre. O Claude Code é nosso parceiro nesse processo — mas a decisão final, o backup, e a validação são humanos.

---

## O fluxograma completo

```
┌─────────────────────────────────────────────────┐
│ 1. PROBLEMA / IDEIA                             │
│    "Quero gerenciar tokens sem ssh na VPS"      │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ 2. PLAN MODE (Claude investiga)                 │
│    - Lê código existente                        │
│    - Identifica o que já existe (reaproveitar)  │
│    - Propõe abordagem por fases                 │
│    - Lista riscos                               │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ 3. REVISÃO HUMANA                               │
│    - Você lê o plano                            │
│    - Pergunta se não entender                   │
│    - Aprova, ajusta ou rejeita                  │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ 4. BACKUP ANTES DE MEXER                        │
│    - cp hub.db hub.db.backup-YYYY-MM-DD         │
│    - (opcional) git branch backup-feature       │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ 5. IMPLEMENTAÇÃO POR FASES                      │
│    Fase 1 = MVP (mínimo que funciona)           │
│    Fase 2 = melhorias                           │
│    Fase 3 = polish                              │
│    Cada fase = commit separado                  │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ 6. TESTE LOCAL                                  │
│    - Build sem erro (npm run build)             │
│    - Sanity check (não tem mensagem assustadora)│
│    - Smoke test no navegador                    │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ 7. DEPLOY (git push + VPS pull)                 │
│    Detalhes: 05-GIT-E-DEPLOY.md                 │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ 8. VALIDAÇÃO EM PRODUÇÃO                        │
│    - pm2 logs (sem erro)                        │
│    - Testar funcionalidade no navegador         │
│    - Pedir feedback de quem usa                 │
└────────────────┬────────────────────────────────┘
                 ↓
        ┌────────┴────────┐
        ↓                 ↓
   FUNCIONOU         QUEBROU
        ↓                 ↓
┌──────────────┐  ┌────────────────────────────┐
│  Atualiza    │  │ 9. ROLLBACK                │
│  doc se      │  │   - git revert HEAD        │
│  precisar    │  │   - git push               │
│  e segue     │  │   - VPS: pull + stop+start │
│  pra próxima │  │   - Restaura DB se mexeu   │
│  fase.       │  │                            │
└──────────────┘  └────────────────────────────┘
```

---

## Princípios não-negociáveis

### 1. Migrations idempotentes
Toda migration de DB precisa ser segura pra rodar **múltiplas vezes**. Usamos `CREATE TABLE IF NOT EXISTS` e `ALTER TABLE ... ADD COLUMN` envoltos em `try { ... } catch {}`.

Por quê: se o servidor reiniciar antes da migration completar, a próxima execução não pode quebrar.

### 2. Fallback sempre
Se o sistema novo (DB) falhar, o sistema velho (.env) precisa funcionar.

Exemplo real: tokens migrados do `.env` pro `app_settings` no DB, mas `getSetting(key)` sempre cai pro `process.env[key]` se DB vazio.

### 3. Backup antes de SQL destrutivo
Comando que envolva `DROP`, `DELETE`, ou `UPDATE` afetando muitas linhas → **backup primeiro, sempre**.

```bash
cp /opt/platform/agency-hub/server/data/hub.db \
   /opt/platform/agency-hub/server/data/hub.db.backup-$(date +%Y%m%d-%H%M)
```

### 4. PM2: stop+start, NÃO restart
`pm2 restart` às vezes não recarrega corretamente o `.env`. Use sempre:
```bash
pm2 stop dros-hub && pm2 start dros-hub
```

(Ou `pm2 restart dros-hub --update-env` se quiser uma linha só. Mas stop+start é mais confiável.)

### 5. Build LOCAL antes de commit (Hub e /core)
Pra Hub e /core, o frontend é buildado no seu PC e a pasta `dist/` vai commitada. Esquecer disso = produção com bundle velho.

```bash
cd agency-hub && npm run build
git add agency-hub/dist/
git commit -m "..."
```

CRM é diferente — build acontece na VPS.

### 6. Commits curtos com prefixo padronizado
```
feat(escopo): descreve o que adiciona
fix(escopo): descreve o que conserta
refactor(escopo): reorganiza sem mudar comportamento
chore(escopo): tarefa de manutenção (build, dist, etc)
docs: documentação
```

Exemplos reais nossos:
- `feat(agency-hub): aba Tokens em Configuracoes`
- `fix(agency-hub): preserva quebras de linha em descricao`
- `chore(agency-hub): -30 nos graficos por funcionario`

### 7. Atualizar doc no mesmo commit da feature
Mudou algo no fluxo? Atualiza `docs/` no **mesmo commit**. Evita doc desatualizada.

---

## Spec-driven, em detalhe

### O que é "spec"?

**Spec = specificação = plano detalhado escrito ANTES de codar.**

Nosso spec sempre tem:
1. **Context** — por que essa mudança? Que problema resolve?
2. **Arquivos a modificar** — lista exata com paths
3. **Detalhes técnicos** — schema do DB, novos endpoints, etc
4. **Edge cases tratados** — tabela de "se acontecer X, comportamento é Y"
5. **Verificação end-to-end** — passos pra testar
6. **Sequência de commits** — cada commit deixa o sistema utilizável
7. **Riscos & mitigações** — o que pode dar errado e como evitar
8. **O que NÃO faz parte do plano** — escopo explícito

### Onde mora o spec?

Quando Claude entra em Plan Mode, ele escreve o spec no arquivo:
```
C:\Users\usuar\.claude\plans\compiled-hugging-deer.md
```

Esse arquivo é **sobrescrito a cada nova tarefa**. Se quiser arquivar um plano antigo, copia ele pra:
```
C:\Users\usuar\.claude\plans\archive-<nome-da-feature>.md
```

Já temos arquivados:
- `archive-tarefas-recorrentes.md` — plano original do sistema de recorrências

### Como funciona Plan Mode

1. Você descreve a tarefa
2. Claude **só lê** (não modifica nada). Explora código relevante via subagents `Explore`.
3. Claude propõe o spec
4. Você lê, dá feedback, ele itera
5. Você aprova → Claude chama `ExitPlanMode`
6. A partir desse momento Claude pode modificar arquivos

> ⚠️ **Nunca aprove plano sem ler.** Mesmo que Claude seja confiável, leitura sua leva 2 min e evita problema.

---

## Implementação por fases

Toda feature grande é dividida em **MVP + iterações**.

### Exemplo real: Aba Configurações (Junho/2026)

**Plano original tinha 3 fases:**

**Fase 1 — Backend de tokens + UI básica**
- Tabela `app_settings`
- Rota `/api/settings` (GET mascarado + PUT)
- Refatorar `Settings.tsx` com tabs (Pipeline + Tokens)
- Componente `TokensTab.tsx`

→ Sistema fica utilizável. User pode gerenciar tokens via UI.

**Fase 2 — Cadastro de contas refinado**
- Aba "Contas do Painel"
- Componente `AccountsTab.tsx`
- Reaproveita `CoreAccountSelect` existente

→ Sistema ganha mais valor. User cadastra contas com search nas APIs.

**Fase 3 — /core dinâmico**
- Endpoint `/api/config/tokens` no Hub
- /core puxa do Hub a cada 10min
- Substitui ALLOWED_CLIENTS hardcoded

→ Sistema fica autônomo. Não precisa mais editar código pra adicionar cliente.

**Cada fase = 1 commit separado.** Se a Fase 2 quebrar algo, dá pra reverter só ela sem perder a Fase 1.

---

## Backup ANTES de mexer

Existe **3 tipos de backup** que você pode/deve fazer:

### 1. Backup do DB
**Sempre** antes de SQL destrutivo:
```bash
cp /opt/platform/agency-hub/server/data/hub.db \
   /opt/platform/agency-hub/server/data/hub.db.backup-$(date +%Y%m%d-%H%M)
```

### 2. Backup via git branch
**Quando** vai fazer refactor grande que pode dar errado:
```bash
git branch backup-before-X
```
Depois pra voltar: `git reset --hard backup-before-X`

### 3. Backup completo (raro)
**Quando** vai mexer em coisa crítica de infra. Snapshot da VPS via HostGator panel.

> 📌 Se o passo seguinte é destrutivo (DROP TABLE, DELETE em massa, força push, etc), pare e pense: "Já tem backup?"

---

## Validação em produção

Depois do deploy, **não saia do terminal antes de validar**.

### Passos mínimos
1. `pm2 status` — todos os processos `online`
2. `pm2 logs <processo> --lines 30 --nostream --err` — sem `Error` ou `Exception`
3. Abre o navegador, vai pra URL afetada, testa o fluxo principal
4. Se tem dado real (cliente recebe email, lead chega, etc), espera 5 min e checa

### Se aparecer erro
1. **Não tenta consertar correndo.** Faz rollback primeiro.
2. Investiga com calma local.
3. Re-deploy com a correção.

---

## Rollback

A coisa mais importante de saber é **como voltar atrás**.

### Rollback de código (git)
```bash
# Local
git revert HEAD                    # cria commit revertendo o último
git push platform master           # sobe

# VPS
cd /opt/platform
git pull origin master
pm2 stop dros-hub && pm2 start dros-hub
```

### Rollback de DB
```bash
# VPS
pm2 stop dros-hub
cp /opt/platform/agency-hub/server/data/hub.db.backup-XXX \
   /opt/platform/agency-hub/server/data/hub.db
pm2 start dros-hub
```

### Rollback completo (código + DB)
Faz os dois acima, em ordem (código primeiro, DB depois).

> ⚠️ **Cuidado:** rollback de DB **descarta dados criados desde o backup**. Se um cliente cadastrou tarefa depois do backup, ela some. Use só se realmente quebrou.

---

## Erros comuns que matamos no nosso processo

| Erro | Como evitamos |
|---|---|
| Token expira sem aviso | Hoje é UI Configurações + tutorial; auto-seed do .env |
| Schema parcial de tabela | Migrations com `try/catch`, detecta + recria se faltar coluna crítica |
| PM2 não pega .env novo | Política: stop+start, não restart |
| Push pro repo errado | Memória do Claude: "platform local, origin upstream" |
| Build esquecido antes de commit | Checklist no spec de Fase: "Build local antes" |
| `==` duplo no .env | Receita de troubleshooting + SQL fix documentado |
| `/core/*` quebrado em Express 5 | Memória do Claude + receita 43 |

---

## Quando NÃO usar esse workflow

Esse processo é pesado pra mudanças triviais. Pra:
- Mudança de texto (string em CSS, label de botão)
- Conserto óbvio de typo
- Adicionar 1 linha de log

Vai direto. Sem Plan Mode. Faz o commit, sobe, valida em 1 min. Sem cerimônia.

> 💡 Regra prática: se a mudança tem **risco zero** e **demora menos de 5 min**, pula o spec. Se demora mais ou tem qualquer chance de quebrar algo, spec primeiro.

---

## Próximo passo

Agora você sabe **o que** fazer. O próximo arquivo te ensina **como subir** as mudanças: [05 — Git e Deploy](05-GIT-E-DEPLOY.md).
