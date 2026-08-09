---
inclusion: manual
description: Gera mensagens commit Conventional Commits concisas focando no why. Ativar ao escrever commits.
---

# Caveman Commit — Mensagens de Commit Compactas

Executar o comando "git commit" com os arquivos em (git) stage.

- Formato Conventional Commits.
- Sem fluff.
- Why over what.
- executar o comando "git commit" com a mensagem gerada.

Apos executar o comando "git commit"
- nao precisa fazer nenhuma explicacao
- somente exibir a mensagem (completa) utilizada no commit
- nao precisa executar o "agent hook" de nome "Lembrar de registrar solução no OKF"

## Regras

**Linha de subject:**
- `<type>(<scope>): <resumo imperativo>` — `<scope>` opcional
- Types: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `chore`, `build`, `ci`, `style`, `revert`
- Modo imperativo: "add", "fix", "remove" — não "added", "adds", "adding"
- <=50 chars quando possível, hard cap 72
- Sem ponto final
- Seguir convenção do projeto para capitalização após o `:`

**Body (só se necessário):**
- Pular quando subject é auto-explicativo
- Adicionar body apenas para: *why* não óbvio, breaking changes, migration notes, issues linkadas
- Wrap em 72 chars
- Bullets `-` não `*`
- Referenciar issues/PRs no final: `Closes #42`, `Refs #17`

**NUNCA incluir:**
- "This commit does X", "I", "we", "now", "currently" — o diff diz o quê
- "As requested by..." — usar trailer Co-authored-by
- "Generated with AI" ou qualquer atribuição AI
- Emoji (a menos que convenção do projeto exija)
- Repetir nome do arquivo quando scope já diz

## Exemplos

Diff: novo endpoint para perfil de usuário
- Errado: "feat: add a new endpoint to get user profile information from the database"
- Certo:
  ```
  feat(api): add GET /users/:id/profile

  Mobile client precisa profile data sem payload completo do user
  para reduzir bandwidth em cold-launch.

  Closes #128
  ```

Diff: breaking API change
- Certo:
  ```
  feat(api)!: rename /v1/orders to /v1/checkout

  BREAKING CHANGE: clients em /v1/orders devem migrar para /v1/checkout
  antes de 2026-06-01. Rota antiga retorna 410 após essa data.
  ```

## Auto-Clareza

Sempre incluir body para: breaking changes, security fixes, data migrations, reverts. Nunca comprimir em subject-only — futuros debuggers precisam do contexto.

## Limites

Apenas gera a mensagem de commit. Não roda `git commit`, não faz stage, não faz amend. Output como bloco de código pronto para colar.
