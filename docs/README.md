# Documentação Dros Agency

> Documentação completa do ecossistema técnico da Dros Agency.
> **Autor:** João Luiz Soares de Mattos
> **Última atualização:** 2026-06-05

---

## Pra quem é essa documentação

- **Funcionário novo** que vai mexer em algum sistema (mesmo sem ser dev)
- **Dono / gerente** que quer entender o que tem disponível
- **Dev futuro** que vai dar continuidade nas evoluções
- **Eu mesmo** daqui a 6 meses quando esquecer alguma coisa

A documentação foi escrita pensando em quem **não tem background técnico**. Termos difíceis aparecem no [Glossário](11-GLOSSARIO.md).

---

## Como navegar

### Ordem recomendada de leitura (~45 min essencial)

1. [01 — Comece Aqui](01-COMECE-AQUI.md) — Seu primeiro contato com a Dros técnica
2. [02 — O Que a Dros Tem](02-O-QUE-A-DROS-TEM.md) — Mapa dos sistemas
3. [03 — Como Usar Claude Code](03-COMO-USAR-CLAUDE-CODE.md) — Nosso jeito de mexer em código
4. [04 — Nosso Workflow](04-NOSSO-WORKFLOW-DEV.md) — Spec-driven com IA, backup e rollback

### Quando precisar mexer em produção

5. [05 — Git e Deploy](05-GIT-E-DEPLOY.md) — Subir mudanças pra VPS
6. [06 — Operação Diária](06-OPERACAO-DIARIA.md) — Comandos do dia a dia
7. [10 — Troubleshooting](10-TROUBLESHOOTING.md) — Problemas comuns e como resolver
8. [12 — Receitas Passo a Passo](12-RECEITAS-PASSO-A-PASSO.md) — Cookbook operacional

### Quando precisar entender uma feature específica

- [07 — Funcionalidades do Hub](07-FUNCIONALIDADES-HUB.md)
- [08 — Funcionalidades do /core](08-FUNCIONALIDADES-CORE.md)
- [09 — Funcionalidades do CRM](09-FUNCIONALIDADES-CRM.md)

### Referência

- [11 — Glossário](11-GLOSSARIO.md) — Dicionário técnico
- [Histórico](HISTORICO.md) — Linha do tempo das implementações

---

## Filosofia de trabalho da Dros

A Dros **não escreve código manualmente**. A gente conversa com a IA (Claude Code) que escreve. Mas isso não é "deixar a IA fazer sozinha" — é um workflow estruturado:

1. **Planejar antes de fazer** — toda mudança começa com um plano detalhado
2. **Backup antes de mexer** — sempre tem como voltar atrás
3. **Implementar por fases** — MVP primeiro, polish depois
4. **Validar em produção** — testar manualmente após cada deploy
5. **Rollback rápido** — se quebrou, voltar é o primeiro passo, não tentar consertar correndo

Detalhes em [04 — Nosso Workflow](04-NOSSO-WORKFLOW-DEV.md).

---

## Avisos importantes

> ⚠️ **Antes de mexer em qualquer DB de produção**: faça backup. Sempre. Sem exceção.
> Veja [Receita 9 — Backup do DB](12-RECEITAS-PASSO-A-PASSO.md#receita-9).

> ⚠️ **Nunca commit `.env` no git**: tokens viraram dado sensível. Hoje estão no DB (via aba Configurações no Hub), mas o `.env` da VPS ainda tem o histórico.

> 💡 **Não tem certeza do que fazer?** Pergunta ao Claude. Em Plan Mode ele investiga e propõe sem fazer nada. Sem risco.

> 📌 **Esta doc fica desatualizada.** Política da Dros: ao mudar algo importante, atualizar a doc no mesmo commit. Veja [04 — Workflow](04-NOSSO-WORKFLOW-DEV.md).

---

## Como contribuir com a doc

Achou um erro? Faltou algo? Reescreveu uma parte?

1. Edita o markdown direto
2. Commit com prefixo `docs:` (ex: `docs: adiciona receita pra resetar senha`)
3. Push pro repo platform

Toda mudança na doc tem o mesmo workflow das mudanças de código — mas sem precisar de plano formal (é texto, não código).
