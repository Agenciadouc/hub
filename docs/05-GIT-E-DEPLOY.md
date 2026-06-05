# 05 — Git e Deploy

> Como salvar código no GitHub e subir as mudanças pra VPS (produção).

---

## Git em 5 minutos (pra leigo)

**Git** é um sistema que salva versões do código. Cada salvamento é um **commit**, e o histórico inteiro fica no GitHub.

### Conceitos

| Termo | O que é |
|---|---|
| **Repo** | O projeto inteiro versionado (pasta + histórico) |
| **Commit** | Um "salvamento" com data, autor e descrição |
| **Branch** | Linha do tempo paralela. A gente usa só `master` |
| **Remote** | Onde o repo mora online (no nosso caso, GitHub) |
| **Push** | Mandar commits locais pro remote |
| **Pull** | Trazer commits do remote pra local |
| **Clone** | Baixar o repo pela primeira vez |

### O ciclo básico

```
1. Você modifica arquivos no PC
2. git add <arquivos>            ← marca o que vai entrar no commit
3. git commit -m "descrição"      ← salva o commit
4. git push <remote> master       ← sobe pro GitHub
```

Pra trazer mudanças que outro dev fez:
```
git pull <remote> master
```

---

## Os 3 remotes da Dros

A pasta `Open Squad` é um **repo umbrella** que contém todos os subprojetos. Mas cada sistema tem seu **próprio repo no GitHub**.

| Sistema | Pasta local | Remote name | URL GitHub |
|---|---|---|---|
| Hub | `agency-hub/` | `platform` | https://github.com/soaresjoaoluiz1/platform |
| /core | `client-dashboard/` | `core` | https://github.com/soaresjoaoluiz1/core |
| CRM | `crm-dashboard/` | `crm` | https://github.com/soaresjoaoluiz1/crm |

> ⚠️ **NÃO use `origin`.** O remote `origin` aponta pro repo upstream (`renatoasse/opensquad`), que é só pra puxar updates do framework Opensquad. **Nunca pushear pra origin.**

### Verificar remotes
```bash
cd c:\Users\usuar\Downloads\Open Squad
git remote -v
```

Saída esperada:
```
core      https://github.com/soaresjoaoluiz1/core.git (fetch)
core      https://github.com/soaresjoaoluiz1/core.git (push)
crm       https://github.com/soaresjoaoluiz1/crm.git (fetch)
crm       https://github.com/soaresjoaoluiz1/crm.git (push)
platform  https://github.com/soaresjoaoluiz1/platform.git (fetch)
platform  https://github.com/soaresjoaoluiz1/platform.git (push)
origin    https://github.com/renatoasse/opensquad.git (fetch)
origin    https://github.com/renatoasse/opensquad.git (push)
```

Se algum estiver faltando: ver [Receita 3](12-RECEITAS-PASSO-A-PASSO.md#receita-3).

---

## Branch única: `master`

A Dros usa só `master`. Sem feature branches, sem develop, sem nada.

Por quê: simplicidade. Equipe pequena, deploy direto, rollback via `git revert`.

> 💡 Se um dia a equipe crescer, adotar git flow ou trunk-based dev. Por enquanto: master only.

---

## Workflow de commit

### 1. Ver o que mudou
```bash
git status
```
Mostra arquivos modificados (não comitados) e arquivos staged (já com `git add`).

### 2. Staged seletivo (recomendado)
Adiciona só os arquivos da feature atual:
```bash
git add agency-hub/server/routes/X.js
git add agency-hub/src/pages/Y.tsx
```

> ⚠️ **Evite `git add -A` ou `git add .`** — pode incluir `.env`, builds esquecidos, lixo. Adicione explicitamente.

### 3. Commit com mensagem padronizada
```bash
git commit -m "feat(agency-hub): adiciona aba Tokens em Configuracoes"
```

Formato:
```
<tipo>(<escopo>): <descrição>

[corpo opcional]
```

Tipos válidos:
- `feat` — feature nova
- `fix` — conserto de bug
- `refactor` — reorganiza sem mudar comportamento
- `chore` — manutenção (build, dist, deps)
- `docs` — documentação

Escopo: `agency-hub`, `core`, `crm`, ou nome do sub-sistema.

### 4. Push pro remote correto
```bash
git push platform master    # pra Hub
git push core master        # pra /core
git push crm master         # pra CRM
```

> 📌 **Não confunda o remote.** Mudança no Hub vai pra `platform`, não pra `core`. Se errar, faz um push pro certo e ignora.

---

## Os 3 fluxos de deploy

Cada sistema tem um fluxo **diferente** de deploy. Importante saber qual.

---

### 🟢 Hub — Frontend buildado LOCAL

**Quando:** mudou `.tsx`/`.ts`/`.css` em `agency-hub/src/`

```bash
# 1. Local — build do frontend
cd c:\Users\usuar\Downloads\Open Squad\agency-hub
npm run build

# 2. Local — commit + push
git add agency-hub/dist/ agency-hub/src/ agency-hub/server/
git commit -m "feat(agency-hub): ..."
git push platform master

# 3. VPS — pull + reinicia processo
ssh root@vps-5269157.3store.com.br
cd /opt/platform
git pull origin master
pm2 stop dros-hub && pm2 start dros-hub
```

> ⚠️ **Se esquecer o `npm run build`** os usuários vão ver o bundle velho. Não vai dar erro, mas as features novas não aparecem.

**Quando NÃO precisa buildar:** mudou só backend (`agency-hub/server/`) sem mexer em `src/`.

---

### 🔵 CRM — Frontend buildado NA VPS

**Quando:** mudou qualquer coisa em `crm-dashboard/`

```bash
# 1. Local — commit + push (sem build local)
git add crm-dashboard/
git commit -m "feat(crm): ..."
git push crm master

# 2. VPS — pull, build NA VPS, reinicia
ssh root@vps-5269157.3store.com.br
cd /root/crm
git pull origin master
cd crm-dashboard
npm run build
cd /root/crm
pm2 stop dros-crm && pm2 start dros-crm
```

Por que diferente do Hub: histórico. CRM começou assim, ainda não migrou.

---

### 🟡 /core — Frontend buildado LOCAL (igual Hub)

**Quando:** mudou qualquer coisa em `client-dashboard/`

```bash
# 1. Local — build + commit + push
cd c:\Users\usuar\Downloads\Open Squad\client-dashboard
npm run build
cd ..
git add client-dashboard/
git commit -m "feat(core): ..."
git push core master

# 2. VPS — pull + reinicia
ssh root@vps-5269157.3store.com.br
cd /root/core
git pull origin master
pm2 stop dros-core && pm2 start dros-core
```

---

## Restart vs Stop+Start no PM2

Sempre prefira **stop+start**:
```bash
pm2 stop dros-hub && pm2 start dros-hub
```

Em vez de:
```bash
pm2 restart dros-hub
```

Por quê: `pm2 restart` às vezes não recarrega corretamente o `.env` ou variáveis em memória. Já nos deu dor de cabeça.

Exceção: se precisa atualizar env e quer comando único:
```bash
pm2 restart dros-hub --update-env
```

> 📌 Memória da Dros: `feedback_pm2_restart_caching.md` — sempre stop+start.

---

## Validação pós-deploy

Toda vez que sobe pra VPS:

```bash
# 1. PM2 está online?
pm2 status

# 2. Sem erros nos primeiros 30s?
sleep 30 && pm2 logs <processo> --lines 20 --nostream --err

# 3. Abre URL no navegador (Ctrl+Shift+R pra hard refresh)

# 4. Testa o fluxo principal da mudança
```

Se errado:
- **Sem pânico.** Faz rollback (próxima seção).
- **Investiga local.** Não tenta consertar correndo na VPS.

---

## Rollback

### Reverter o último commit (cria commit novo)
```bash
# Local
git revert HEAD
git push platform master

# VPS
ssh root@vps-...
cd /opt/platform
git pull origin master
pm2 stop dros-hub && pm2 start dros-hub
```

> 💡 `git revert` cria um commit que **desfaz** o anterior. Histórico preservado.

### Voltar pra um commit específico (destrutivo)
```bash
# Local — só se SOZINHO no branch
git reset --hard <commit-hash>
git push platform master --force
```

> ⚠️ `--force` é perigoso. Reescreve histórico. Use só se ninguém mais tá puxando do repo.

### Rollback de DB
Ver [Receita 39](12-RECEITAS-PASSO-A-PASSO.md#receita-39).

---

## Acessos GitHub

Pra pushear, você precisa de:
1. Conta GitHub com permissão de write nos repos da Dros
2. Token de acesso pessoal (PAT) ou SSH key cadastrada

### Configurar PAT (HTTPS)
1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token
2. Permissões: `repo` (read+write)
3. Copia o token (só aparece uma vez)
4. Na primeira vez que pushear, o GitHub pede usuário + token. Cola.
5. Windows lembra (Credential Manager). Mac via Keychain.

### Configurar SSH key
```bash
ssh-keygen -t ed25519 -C "seu@email.com"
cat ~/.ssh/id_ed25519.pub
```
Cola o conteúdo em GitHub → Settings → SSH and GPG keys → New SSH key.

Depois, troca a URL do remote pra SSH:
```bash
git remote set-url platform git@github.com:soaresjoaoluiz1/platform.git
```

---

## Comuns erros e como resolver

### "Permission denied (publickey)"
SSH key não cadastrada no GitHub ou inválida. Refaça o passo da SSH key acima.

### "fatal: refusing to merge unrelated histories"
Aconteceu confusão de remotes. Resolve com:
```bash
git pull <remote> master --allow-unrelated-histories
```
Resolve conflitos se aparecerem, depois `git push`.

### "rejected (non-fast-forward)"
Alguém pushou antes de você. Faz pull primeiro:
```bash
git pull <remote> master
# resolve conflito se tiver
git push <remote> master
```

### "remote: Repository not found"
URL do remote errada (ou repo privado sem permissão). Confere `git remote -v`.

---

## Boas práticas que a gente segue

- ✅ Commit pequeno e atômico (uma feature por commit, não 10)
- ✅ Mensagem clara (você + 6 meses entende?)
- ✅ Sempre push depois de commit (não acumula local)
- ✅ Pull antes de começar a mexer (evita conflito)
- ❌ Nunca commit `.env`
- ❌ Nunca force push em master sem avisar
- ❌ Nunca commit credenciais ou tokens em texto puro
- ❌ Nunca usar `git add -A` sem revisar `git status` antes

---

## Próximo passo

Agora você sabe deploy. Próximo: comandos do dia a dia → [06 — Operação Diária](06-OPERACAO-DIARIA.md).
