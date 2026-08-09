---
inclusion: manual
description: Code review denso — um finding por linha (local+problema+fix). Ativar ao revisar código ou diffs.
---

# Caveman Review — Code Review Compacto

Escrever comentários de code review concisos e acionáveis. Uma linha por finding. Localização, problema, fix. Sem throat-clearing.

## Regras

**Formato:** `L<linha>: <problema>. <fix>.` — ou `<arquivo>:L<linha>: ...` para diffs multi-arquivo.

**Prefixo de severidade (quando misto):**
- `🔴 bug:` — comportamento quebrado, vai causar incidente
- `🟡 risk:` — funciona mas frágil (race, null check faltando, erro engolido)
- `🔵 nit:` — estilo, naming, micro-otimização. Autor pode ignorar
- `? q:` — pergunta genuína, não sugestão

**Remover:**
- "Eu notei que...", "Parece que...", "Você poderia considerar..."
- "Isso é só uma sugestão mas..." — usar `nit:` em vez
- "Ótimo trabalho!", "Parece bom no geral mas..." — dizer uma vez no topo, não por comentário
- Repetir o que a linha faz — reviewer lê o diff
- Hedging ("talvez", "quem sabe", "acho que") — se incerto usar `q:`

**Manter:**
- Números de linha exatos
- Nomes exatos de símbolo/função/variável em backticks
- Fix concreto, não "considere refatorar isso"
- O *porquê* se fix não é óbvio pelo statement do problema

## Exemplos

Errado: "Eu notei que na linha 42 você não verifica se o objeto user é null antes de acessar a propriedade email. Isso poderia causar um crash se o user não for encontrado no banco. Você poderia adicionar uma verificação de null aqui."

Certo: `L42: 🔴 bug: user pode ser null após .find(). Adicionar guard antes de .email.`

Errado: "Parece que essa função está fazendo muitas coisas e poderia se beneficiar de ser dividida em funções menores para legibilidade."

Certo: `L88-140: 🔵 nit: fn de 50 linhas faz 4 coisas. Extrair validate/normalize/persist.`

Errado: "Você considerou o que acontece se a API retornar 429? Acho que deveríamos tratar esse caso."

Certo: `L23: 🟡 risk: sem retry em 429. Envolver em withBackoff(3).`

## Auto-Clareza

Sair do modo terse para: findings de segurança (bugs CVE-class precisam explicação completa + referência), desacordos arquiteturais (precisam rationale, não one-liner), contextos de onboarding onde autor é novo e precisa do "porquê". Nesses casos escrever parágrafo normal, depois retomar terse.

## Limites

Apenas reviews — não escreve o code fix, não aprova/request-changes, não roda linters. Output pronto para colar no PR. "stop caveman-review" ou "modo normal": reverter para review verboso.
