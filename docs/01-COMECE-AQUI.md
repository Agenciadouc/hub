# 01 — Comece Aqui

> Esse é o primeiro arquivo a ler. Se você nunca mexeu em nada técnico da Dros, leia daqui e siga a ordem dos próximos arquivos.

---

## Bem-vindo

Você está na documentação técnica da **Dros Agency**. A Dros não é só uma agência de marketing — ela construiu seus próprios sistemas internos. Isso significa que parte do seu trabalho pode envolver entender, usar ou até modificar esses sistemas.

**Calma:** você não precisa ser programador. A gente usa **IA (Claude Code)** pra escrever o código. Seu papel é entender o que quer, conversar com a IA, revisar o que ela propõe, e validar o resultado.

---

## O que a Dros tem de tecnológico

Três sistemas principais rodando 24/7 num servidor (VPS):

### 🟢 Hub — `drosagencia.com.br/hub`
**O cérebro da agência.** Aqui acontece tudo:
- Pipeline de tarefas (Kanban estilo Trello)
- Aprovações com cliente
- Painel de Performance (Meta Ads, Google Ads, Analytics, Instagram)
- Cadastro de clientes
- Equipe e departamentos
- Financeiro
- Configurações (incluindo tokens das integrações)

> Quem usa: dono, gerentes, funcionários, clientes (com acesso limitado).

### 🟡 /core — `drosagencia.com.br/core`
**Painel Performance standalone.** Versão "raiz" do painel de métricas, antes de tudo virar parte do Hub. Hoje funciona em paralelo, pra clientes que ainda usam essa URL ou pra acessar via embed.

> Quem usa: time interno principalmente. Cliente raramente.

### 🔵 CRM — `drosagencia.com.br/crm`
**Gerenciador de leads do WhatsApp.** Integrado com Evolution API, conversa com leads vindos dos sites de venda. Tem atendentes humanos e agentes de IA.

> Quem usa: equipe comercial / SDR.

---

## Como você vai interagir com tudo isso

### Cenário 1: você é **funcionário operacional** (vai usar, não modificar)
- Lê esta doc até o arquivo 04 (workflow)
- Pula direto pros catálogos de funcionalidades (07, 08, 09)
- Quando precisar, consulta [Receitas](12-RECEITAS-PASSO-A-PASSO.md)

### Cenário 2: você é **gerente** (precisa entender, configurar)
- Lê esta doc até o arquivo 05 (deploy)
- Lê o catálogo do sistema que você mais usa (07, 08 ou 09)
- Sabe acionar [Receitas Operacionais](12-RECEITAS-PASSO-A-PASSO.md) (renovar token, etc)

### Cenário 3: você é **dev** (vai modificar código)
- Lê tudo na ordem 01 → 06
- Foca nos arquivos 03 (Claude Code) e 04 (Workflow)
- Mantém a [HISTORICO](HISTORICO.md) atualizada quando entregar algo

---

## Pré-requisitos antes de começar

Pra **operar** os sistemas (só usar): nada além do navegador e suas credenciais.

Pra **modificar código**:

### 1. Acessos necessários
- [ ] Login GitHub com permissão nos repos da Dros (peça pro João Luiz)
- [ ] Acesso SSH à VPS HostGator (chave + IP + usuário root)
- [ ] Login admin no Hub (`drosagencia.com.br/hub`)
- [ ] Login admin no /core (`admin@drosagencia.com.br`)

### 2. Softwares instalados no seu PC
- [ ] **Git** — pra baixar/subir código (https://git-scm.com/)
- [ ] **Node.js 18+** — pra rodar builds locais (https://nodejs.org/)
- [ ] **Claude Code** — a IA que escreve código (instalar via terminal)
- [ ] **VS Code** ou outro editor de código
- [ ] **Cliente SSH** — Windows tem nativo, Mac também

### 3. Clonar o repo principal
```bash
git clone https://github.com/soaresjoaoluiz1/platform.git
cd platform
```

Detalhes da instalação completa: [Receita 1](12-RECEITAS-PASSO-A-PASSO.md#receita-1).

---

## Conceito-chave: a gente NÃO escreve código direto

Esse é o ponto mais importante de toda essa documentação.

**Tradicional:** dev abre o editor, digita código, salva, testa.

**Dros (com Claude Code):**
1. Dev abre o terminal e roda `claude`
2. Dev **conversa em português** com a IA: "Quero adicionar uma aba de configurações no Hub onde possa gerenciar tokens"
3. Claude lê o código existente, propõe um plano detalhado
4. Dev revisa o plano (aprova, ajusta, ou recusa)
5. Claude implementa o código
6. Dev testa, valida, sobe pra produção

> 💡 **Por que isso é melhor?**
> - Mais rápido (10x menos tempo digitando)
> - Mais seguro (Claude lê o código todo antes de mexer, não chuta)
> - Mais acessível (você não precisa saber sintaxe de JavaScript, só explicar o que quer)
> - Documentado (cada mudança vira um plano + commit estruturado)

Como funciona na prática? Próximo arquivo: [03 — Como Usar Claude Code](03-COMO-USAR-CLAUDE-CODE.md).

---

## Mapa mental do ecossistema

```
        ┌───────────────────────┐
        │   Cliente final       │
        │  (acessa via browser) │
        └──────────┬────────────┘
                   ↓
        ┌───────────────────────────────────────┐
        │ Cloudflare + Apache (drosagencia.com.br)│
        └──────────┬────────────────────────────┘
                   ↓
   ┌───────────────┼───────────────┐
   ↓               ↓               ↓
 /hub            /core            /crm
 :3003           :3004            :3002
 (Hub)          (Painel Perf)    (CRM Whats)
   │               │               │
   └──────┬────────┴───────┬───────┘
          ↓                ↓
    ┌──────────┐    ┌──────────┐
    │ SQLite   │    │ Evolution│
    │ hub.db   │    │ (Docker) │
    └──────────┘    └──────────┘
          │
          ↓
    APIs externas:
    - Meta Graph
    - Google Ads
    - GA4
    - Kiwify
```

Cada sistema é um **processo Node.js** rodando via **PM2** (gerenciador de processos) na VPS.

---

## Próximo passo

Vai pra [02 — O Que a Dros Tem](02-O-QUE-A-DROS-TEM.md) pra ver detalhes de cada sistema.
