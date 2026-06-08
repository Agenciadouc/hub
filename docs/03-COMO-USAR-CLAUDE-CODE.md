# 03 — Como Usar Claude Code

> O Claude Code é a ferramenta principal da Dros pra mexer em código. Esse arquivo te ensina a usar bem.

---

## O que é Claude Code

**Claude Code** é um assistente de IA da Anthropic que roda no seu terminal. Ele:
- Lê e entende todo o código do projeto
- Conversa com você em português
- Propõe planos antes de fazer mudanças grandes
- Executa as mudanças (edita arquivos, commita, sobe pra produção)
- Aprende seu jeito de trabalhar (memória persistente)

> 💡 **Claude Code ≠ ChatGPT.** ChatGPT é um chat genérico. Claude Code mora no terminal junto com seu código, lê arquivos reais, executa comandos reais. Não inventa.

---

## Instalação

### Windows / Mac
```bash
npm install -g @anthropic-ai/claude-code
```

Ou via instaladores oficiais: https://www.anthropic.com/claude-code

### Primeira execução
```bash
claude
```
Vai pedir login Anthropic na primeira vez (browser abre automático). Depois fica logado.

> ⚠️ A conta Anthropic da Dros precisa ter assinatura ativa (Pro ou Max). Sem isso, Claude Code não funciona.

---

## Como abrir no projeto

Sempre rode `claude` **dentro da pasta do projeto** que você quer mexer:

```bash
cd c:\Users\usuar\Downloads\Open Squad
claude
```

Isso é importante porque o Claude vai indexar essa pasta como contexto. Se você rodar no lugar errado, ele não vai achar os arquivos.

---

## Os 3 modos de operação

### 1. Modo padrão (sem flag)
Claude lê, propõe e executa. Pergunta antes de ações destrutivas.

Use quando: já sabe o que quer e a tarefa é simples.

### 2. Plan Mode 🔵 (Shift+Tab → Plan Mode)
Claude **só investiga e propõe um plano**. Não modifica nada até você aprovar.

Use quando: mudança grande, você quer ver o plano antes.

Como acionar:
```
Aperta Shift+Tab até aparecer "plan mode" no rodapé do terminal.
```

Quando Claude termina de planejar, ele chama `ExitPlanMode` mostrando o plano completo. Você:
- **Aprova** → ele executa
- **Rejeita** → pede ajuste
- **Edita o plano** → modifica direto e aprova

### 3. Auto Mode 🟠 (Shift+Tab → Auto Mode)
Claude **não pergunta nada**. Faz tudo direto.

Use quando: tarefa repetitiva, você confia 100% no fluxo.

> ⚠️ **Cuidado com Auto Mode.** Use só quando o escopo está claro e você revisou o plano antes. Senão pode quebrar algo sem te avisar.

---

## Fluxo recomendado pra mudanças grandes

```
1. Abre Claude no projeto (`claude`)
2. Shift+Tab → Plan Mode
3. Descreve a tarefa em português ("Quero adicionar tab X em Y...")
4. Claude investiga (lê código, pergunta dúvidas se precisar)
5. Claude propõe plano com fases
6. Você revisa: dá feedback ou aprova
7. Aprovado → Claude sai do Plan Mode e executa
8. Você acompanha (Claude vai mostrando cada arquivo modificado)
9. Build local + commit + push (Claude pode fazer)
10. Deploy na VPS (você executa os comandos que Claude mostrar)
```

---

## Comandos especiais

### Built-in do Claude Code
- `/help` — ajuda
- `/clear` — limpa o contexto atual (começa conversa nova)
- `/cost` — quanto você gastou de tokens
- `/model` — troca o modelo (sonnet vs opus etc)

### Específicos da Dros
- `/opensquad` — menu do framework Opensquad (squads pra tarefas específicas)
- `/loop <tarefa>` — Claude trabalha em loop até completar
- `/ultrareview` — multi-agent review de uma branch/PR (cobrado à parte)

> 📌 `/ultrareview` é pago e disparado por você. Não confundir com Claude rodar review sozinho.

---

## Memória persistente

Claude Code guarda **memórias entre conversas** em:
```
C:\Users\usuar\.claude\projects\<nome-do-projeto>\memory\
```

São arquivos markdown com fatos sobre você, sobre o projeto, sobre como prefere trabalhar. O Claude lê automaticamente no início de cada conversa.

### Tipos de memória

| Tipo | O que é | Exemplo |
|---|---|---|
| `user` | Quem é você | "João Luiz, dono da Dros, prefere respostas concisas" |
| `feedback` | Como trabalhar com você | "Sempre buildar local antes de commitar Hub" |
| `project` | Estado do projeto | "Hub usa Node 16 na VPS, frontend buildado local" |
| `reference` | Onde achar coisas | "Logs ficam em pm2 logs <processo>" |

> 💡 Você pode pedir explicitamente: "Lembre disso: X". Claude salva.
> Ou "Esquece X": Claude apaga.

### Memórias atuais da Dros (parcial)

- `feedback_grep_all_occurrences.md` — sempre grepar todas ocorrências antes de fix
- `feedback_no_claude_coauthor.md` — não adicionar Co-Authored-By: Claude em commits
- `feedback_ship_everything.md` — em deploys, subir tudo do working tree
- `feedback_pm2_restart_caching.md` — usar pm2 stop+start, não restart
- `reference_hub_deploy.md` — fluxo de deploy do Hub
- `reference_core_deploy.md` — fluxo de deploy do /core
- `reference_crm_deploy.md` — fluxo de deploy do CRM
- `project_performance_migration.md` — Performance migrada de /core pra Hub

Lista completa: ver pasta `memory/`.

---

## Exemplos de prompts: bons vs ruins

### ❌ Ruim
> "Conserta o bug"

Por que: zero contexto. Claude vai chutar.

### ✅ Bom
> "No Hub, quando arrasto a tarefa do pipeline pra outra coluna, o modal abre. Ver Pipeline.tsx, eu acho que é o onDragEnd. Quero que o drag NÃO abra o modal."

Por que: aponta o arquivo, descreve o sintoma e o efeito desejado.

---

### ❌ Ruim
> "Faz uma feature de relatórios"

Por que: vago, escopo gigante.

### ✅ Bom
> "Quero uma página `/hub/relatorios` (admin only) com filtros por cliente e período. Cards: total de tarefas concluídas, tempo médio de produção, taxa de retrabalho. Exportável pra Excel. Vamos no Plan Mode primeiro."

Por que: define rota, role, métricas, exportação, e pede plano antes.

---

### ❌ Ruim
> "Sobe pra produção"

Por que: o quê? Que branch? Qual sistema?

### ✅ Bom
> "Sobe a feature de aba Configurações pra produção. Build local primeiro (já fiz commit), push pro platform master, depois me dá os comandos pra rodar na VPS."

Por que: especifica o sistema, divisão de responsabilidades (Claude empurra código, você executa SSH).

---

## Boas práticas

### Antes de pedir

1. **Tenha clareza do que quer** — escreva em 1 frase. Se não consegue, é sinal que precisa pensar mais.
2. **Identifique o contexto** — qual sistema (Hub/Core/CRM)? Qual página? Qual arquivo?
3. **Saiba o "porquê"** — Claude implementa melhor quando entende a motivação, não só a ação.

### Durante a conversa

1. **Em Plan Mode, leia o plano todo antes de aprovar.**
2. **Pergunte se tiver dúvida.** Claude explica.
3. **Não confie 100%.** Olhe o diff dos arquivos modificados.
4. **Se quebrou, peça pra reverter.** Claude sabe fazer git revert.

### Depois

1. **Teste local** — Claude faz build, mas você abre no navegador.
2. **Deploy é etapa separada** — sempre comandos explícitos na VPS.
3. **Atualize a doc** se mudou algo importante (ver [04 — Workflow](04-NOSSO-WORKFLOW-DEV.md)).
4. **Pede pra Claude salvar memória** se aprendeu algo novo sobre o projeto.

---

## Limitações do Claude Code

- **Não acessa internet por padrão.** Tem ferramentas (`WebFetch`, `WebSearch`) mas precisa pedir.
- **Não roda código com efeitos colaterais externos sem permissão** — ele pede antes de mandar email, fazer commit, etc.
- **Limites de contexto.** Se a conversa fica muito longa, ele resume e pode esquecer detalhes. Use `/clear` periodicamente.
- **Pode alucinar.** Verifique sempre o diff antes de commitar.

---

## E se o Claude Code não estiver disponível?

Você ainda tem acesso ao código via VS Code. Mas perde:
- A velocidade da IA
- O Plan Mode estruturado
- A memória persistente

Pra mexer manualmente, leia [05 — Git e Deploy](05-GIT-E-DEPLOY.md) e use VS Code direto.

---

## Próximo passo

Agora que sabe usar a ferramenta, vamos pro **workflow Dros**: [04 — Nosso Workflow](04-NOSSO-WORKFLOW-DEV.md).
