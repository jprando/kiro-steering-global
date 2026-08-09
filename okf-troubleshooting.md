---
inclusion: always
description: "Consulta knowledge/ antes de resolver problemas e registra soluções não-triviais no OKF (só ativa se knowledge/ existir no workspace)."
---

# OKF — Consulta e Registro de Soluções

> **Pré-condição obrigatória:** Este steering só se aplica se a pasta `knowledge/` existir na raiz do workspace aberto. Se `knowledge/` **não existir**, IGNORAR COMPLETAMENTE todas as instruções abaixo — não consultar, não registrar, não criar a pasta. Abortar imediatamente.

---

Quando o code agent enfrentar um problema, erro, ou for solicitada ajuda para resolver algo:

## 1. Consultar OKF primeiro

Antes de tentar resolver por conta própria, verificar se `knowledge/` contém orientações relevantes:

1. Consultar `knowledge/operacao/troubleshooting.md` — problemas comuns e soluções conhecidas.
2. Se o problema for de outra área, navegar via `knowledge/index.md` para localizar conceitos relevantes (arquitetura, infra, features, seguranca, convencoes).
3. Usar grep/search na pasta `knowledge/` com palavras-chave do erro ou problema.

Se o OKF tiver a solução: aplicar diretamente, citando a fonte.

## 2. Registrar soluções não-triviais no OKF

Seguir #[[steering:global:/home/jeudi/.kiro/steering/okf-knowledge-base.md]] §4.5 (Ao registrar solução de troubleshooting) — critérios, formatos e procedimento completo estão lá.

### 2.1 Cross-referência nos conceitos (Problemas Conhecidos)

Além de registrar no `troubleshooting.md`, o agente DEVE cross-referenciar nos arquivos de conceito relevantes. Objetivo: dar visibilidade aos troubleshooting conhecidos no local onde o desenvolvedor naturalmente consulta.

#### Procedimento ao registrar troubleshooting

1. **Registrar no `troubleshooting.md`** — continua sendo ponto central de indexação. Cada entrada DEVE ter um ID estável no formato `TS-NNN` (incremental) usado como anchor (`### TS-001 — Título curto`).
2. **Identificar conceitos relacionados** — determinar quais arquivos em `knowledge/` (arquitetura, convencoes, decisoes, dominio, features, frontend, infra, operacao) são contexto direto do problema.
3. **Adicionar link no conceito** — no arquivo de conceito identificado, adicionar/atualizar seção `## Problemas Conhecidos` (sempre no final do documento, antes de `# Citações` se existir). Formato:
   ```markdown
   ## Problemas Conhecidos

   - Descrição curta em uma frase → [TS-001](../operacao/troubleshooting.md#ts-001)
   - Outro problema do mesmo conceito → [TS-007](../operacao/troubleshooting.md#ts-007)
   ```
4. **Bidirecionalidade no troubleshooting.md** — cada entrada no troubleshooting DEVE incluir campo `**Conceitos**:` listando links para os arquivos de conceito que referenciam esse problema:
   ```markdown
   ### TS-001 — Título curto do problema

   **Conceitos**: [nome-conceito](../path/conceito.md), [outro](../path/outro.md)
   **Causa**: Explicação da causa raiz.
   **Solução**: Passos para resolver.
   ```

#### Regras

- Só linkar conceito que tenha relação direta com o problema. Não forçar link se o troubleshooting é puramente operacional e nenhum conceito existente é contexto relevante.
- Se conceito já tem seção `## Problemas Conhecidos`, apenas adicionar nova linha à lista existente.
- Se conceito não tem a seção, criá-la.
- Um troubleshooting pode referenciar múltiplos conceitos (e vice-versa).
- IDs são estáveis — nunca reusar ID de entrada removida.
