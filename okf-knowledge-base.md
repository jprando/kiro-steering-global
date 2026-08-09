---
inclusion: always
description: Mantenedor de knowledge base OKF — regras de ingest, cross-linking, conformance e workflows para bundles knowledge/
---

# OKF — Open Knowledge Format

Steering para manutenção de knowledge bases no formato OKF.
Quando ativado, o agente opera como **mantenedor de wiki**: ingere fontes, cria/atualiza conceitos, mantém cross-references, e preserva a base consistente.

---

## 1. Afinidade — O Padrão LLM Wiki

O OKF implementa o padrão **LLM Wiki** (Andrej Karpathy, 2025): em vez de re-derivar conhecimento a cada query via RAG, o LLM **constrói e mantém incrementalmente** uma wiki persistente — artefato composto que acumula valor a cada fonte adicionada.

### Arquitetura em 3 camadas

1. **Fontes brutas** (`raw/` ou externas) — documentos imutáveis. Artigos, specs, dumps, transcrições. LLM lê, nunca modifica.
2. **Wiki/Bundle OKF** (`knowledge/`) — markdown gerado e mantido pelo LLM. Conceitos, resumos, cross-references. LLM é dono desta camada.
3. **Schema/Steering** (este arquivo) — convenções, workflows, regras. Co-evoluído entre humano e LLM.

### Operações fundamentais

- **Ingest** — Nova fonte chega. LLM lê, extrai conhecimento, cria/atualiza conceitos no bundle, atualiza `index.md`, registra em `log.md`. Uma fonte pode tocar 5-15 páginas.
- **Query** — Pergunta contra o bundle. LLM consulta `index.md` para localizar conceitos relevantes, lê, sintetiza resposta. Boas respostas podem virar novos conceitos.
- **Lint** — Health-check periódico. Busca: contradições, claims obsoletos, orphan pages sem inbound links, conceitos mencionados mas sem página própria, cross-references faltantes.

### Princípio central

> O trabalho tedioso de manter uma knowledge base não é ler ou pensar — é bookkeeping. Atualizar cross-references, manter resumos atuais, notar contradições. Humanos abandonam wikis porque custo de manutenção cresce mais rápido que valor. LLM elimina esse custo.

**Papel do humano:** curar fontes, direcionar análise, fazer perguntas certas.
**Papel do LLM:** todo o resto — resumir, cross-referenciar, arquivar, manter consistência.

---

## 2. Motivação — Por que OKF

OKF é formato **universal e vendor-neutral** para representar conhecimento como markdown + YAML frontmatter.

### Propriedades que justificam a escolha

- **Human- e agent-readable** — sem SDK. `cat` lê, LLM ingere verbatim.
- **Version-controllable** — vive em git. PRs, diffs, blame funcionam. Curadoria de conhecimento = atividade normal de engenharia.
- **Portável e lock-in free** — bundle é diretório. Tarball, repo, filesystem, qualquer sistema que fala arquivos.
- **Estruturado + não-estruturado** — frontmatter para campos queryáveis (`type`, `resource`, `tags`, `timestamp`); body markdown para prosa, schemas, exemplos que LLMs leem.
- **Minimamente opinado** — poucas chaves obrigatórias, extensível livremente.
- **Compõe com tooling existente** — Obsidian, Notion, MkDocs, Hugo, Jekyll — todos falam markdown + YAML frontmatter.
- **Progressive disclosure** — `index.md` permite navegar hierarquia um nível por vez sem carregar bundle inteiro em contexto.
- **Graph-shaped** — links entre conceitos via markdown links normais expressam relações além da hierarquia de diretórios.

---

## 3. Especificação — OKF v0.1

> **Spec canônica:** Se `knowledge/spec.md` existir no bundle, ele contém a especificação completa e autoritativa do OKF. Nesse caso, priorizar `knowledge/spec.md` sobre o resumo abaixo para questões de formato, conformance e semântica. O resumo abaixo serve como quick-reference operacional.

### 3.1 Bundle Structure

Bundle = árvore de diretórios com arquivos markdown.

**Diretório padrão: `knowledge/`** na raiz do projeto. Quando nenhum path for especificado, SEMPRE usar `knowledge/` como diretório do bundle.

```
knowledge/
├── index.md              # Opcional. Listagem progressiva.
├── log.md                # Opcional. Histórico cronológico.
├── <conceito>.md         # Conceito na raiz.
└── <subdiretorio>/       # Agrupamento lógico.
    ├── index.md
    ├── <conceito>.md
    └── <subdiretorio>/
```

Distribuição: git repo (recomendado), tarball, subdiretório de repo maior.

### 3.2 Reserved filenames

| Arquivo | Propósito |
|---------|-----------|
| `index.md` | Listagem de diretório (§3.5) |
| `log.md` | Histórico de atualizações (§3.6) |

Todos outros `.md` = conceitos.

### 3.3 Concept Documents

Todo conceito = UTF-8 markdown com 2 partes:

1. **YAML frontmatter** (entre `---`)
2. **Body** (markdown livre)

#### Frontmatter

```yaml
---
type: <Tipo>                       # OBRIGATÓRIO
title: <Nome display>              # Recomendado
description: <Resumo 1 linha>      # Recomendado
resource: <URI canônica>           # Opcional (ausente para conceitos abstratos)
tags: [tag1, tag2]                 # Opcional
timestamp: <ISO 8601>             # Opcional (última mudança significativa)
# ... chaves adicionais livres
---
```

**Obrigatório:** `type` — string curta identificando tipo do conceito. Valores não registrados centralmente. Exemplos: `Tabela`, `API Endpoint`, `Métrica`, `Playbook`, `Referência`, `Componente`, `Conceito`.

**Extensões:** chaves adicionais permitidas livremente. Consumers preservam chaves desconhecidas.

#### Body

Markdown padrão. Preferir estrutura (headings, listas, tabelas, code blocks) sobre prosa livre — estrutura ajuda retrieval.

Headings convencionais:

| Heading | Propósito |
|---------|-----------|
| `# Schema` | Descrição estruturada de colunas/campos |
| `# Exemplos` | Exemplos concretos (code blocks) |
| `# Citações` | Fontes externas que sustentam claims |

### 3.4 Cross-linking

Links entre conceitos via markdown links **sempre relativos ao arquivo que contém o link**.

> ⚠️ **NUNCA usar links com `/` inicial** (ex: `/duckdb.md`, `/arquitetura/seguranca.md`). Links com `/` resolvem para a **raiz do projeto** (ou raiz do filesystem), não para a raiz do bundle `.okf/`. Isso quebra navegação em IDEs, GitHub, e qualquer renderer markdown.

**Arquivo na raiz do bundle** linkando para outro na raiz:
```markdown
Ver [DuckDB](duckdb.md) para detalhes do motor.
Ver [Segurança](arquitetura/seguranca.md) para camadas de proteção.
```

**Arquivo em subdiretório** linkando para a raiz ou outro subdiretório:
```markdown
Ver [DuckDB](../duckdb.md) para configuração.
Ver [Streaming](../arquitetura/streaming-sse.md) para SSE.
Ver [conceito vizinho](outro-conceito.md) no mesmo diretório.
```

**Regra prática:**
- Mesmo diretório → `(arquivo.md)`
- Subdiretório abaixo → `(subdir/arquivo.md)`
- Diretório acima → `(../arquivo.md)`
- Diretório irmão → `(../irmao/arquivo.md)`

Link de A para B = **relacionamento dirigido**. Tipo do relacionamento inferido pela prosa ao redor (não pelo link em si).

Consumers DEVEM tolerar links quebrados (target inexistente = conhecimento ainda não escrito).

### 3.5 Index Files

`index.md` pode aparecer em qualquer diretório. Enumera conteúdo para **progressive disclosure**.

Sem frontmatter. Body usa seções com listas:

```markdown
# Seção / Grupo

* [Título 1](url-relativa-1) - descrição curta
* [Título 2](url-relativa-2) - descrição curta

# Outra Seção

* [Subdiretório](subdir/) - descrição do subdiretório
```

Entries devem incluir description do frontmatter do conceito linkado.

### 3.6 Log Files

`log.md` = histórico de mudanças. Lista de entradas agrupadas por data, mais recente primeiro:

```markdown
# Log de Atualizações

## 2026-07-06
* **Criação**: Conceito [X](path/x.md) criado a partir de fonte Y.
* **Atualização**: [Z](path/z.md) — adicionado schema atualizado.

## 2026-07-05
* **Inicialização**: Estrutura base do bundle criada.
```

Date headings: ISO 8601 `YYYY-MM-DD`. Palavra bold inicial é convenção (`**Criação**`, `**Atualização**`, `**Depreciação**`, `**Remoção**`).

### 3.7 Citações

Claims sourced de material externo listados sob `# Citações`, numerados:

```markdown
# Citações

[1] [Título da fonte](https://url-da-fonte)
[2] [Outra fonte](https://outra-url)
```

### 3.8 Conformance

Bundle conformante OKF v0.1 se:
1. Todo `.md` não-reservado tem YAML frontmatter parseável.
2. Todo frontmatter contém `type` não-vazio.
3. Arquivos reservados (`index.md`, `log.md`) seguem estrutura definida.

Consumers NÃO rejeitam bundle por: campos opcionais faltando, `type` desconhecido, chaves extras, links quebrados, `index.md` ausente.

---

## 4. Workflows para o Agente

### 4.1 Ao criar novo bundle

1. Criar diretório `knowledge/` na raiz do projeto (ou path explicitamente especificado).
2. Criar `knowledge/index.md` com listagem vazia ou inicial.
3. Criar `knowledge/log.md` com entrada de inicialização.

### 4.2 Ao ingerir fonte

1. Ler fonte completa.
2. Identificar conceitos a criar/atualizar.
3. Para cada conceito novo: criar `.md` com frontmatter completo + body estruturado.
4. Para cada conceito existente: atualizar body/frontmatter preservando cross-links.
5. Atualizar `index.md` dos diretórios afetados.
6. Append entrada em `log.md` com data atual.
7. Verificar cross-links: novos conceitos devem linkar para relacionados e vice-versa.

### 4.3 Ao responder query

1. Ler `index.md` raiz para localizar conceitos relevantes.
2. Ler conceitos identificados.
3. Sintetizar resposta com citações aos conceitos.
4. Se resposta gerou insight novo valioso: propor criação como conceito no bundle.

### 4.4 Ao fazer lint

1. Listar todos `.md` do bundle.
2. Verificar: frontmatter parseável com `type` presente.
3. Identificar: links quebrados, orphan pages, conceitos mencionados sem página.
4. Reportar findings e propor correções.

### 4.5 Ao registrar solução de troubleshooting

Após resolver um problema **não-trivial** (exigiu investigação, múltiplas tentativas, workaround, configuração não-óbvia, ou erro críptico), registrar no bundle.

#### Critérios para registrar

- Problema exigiu mais de 1 tentativa para resolver
- Solução envolve configuração não-óbvia ou workaround
- Erro tem mensagem críptica que dificulta diagnóstico
- Problema pode recorrer (não é one-off)
- Solução contradiz intuição ou documentação oficial

#### Critérios para NÃO registrar

- Typo ou erro de sintaxe trivial
- Import faltando (auto-resolvido)
- Problema já documentado no bundle
- Erro causado por código ainda em desenvolvimento ativo (WIP)

#### Procedimento

1. **Inline** — se encaixa em `knowledge/operacao/troubleshooting.md`: adicionar seção seguindo formato:
   ```markdown
   ### Título curto do problema

   **Causa**: Explicação da causa raiz.
   **Solução**: Passos para resolver.
   ```
2. **Standalone** — conceito novo que merece documento próprio: criar em diretório apropriado com `type: Playbook` (troubleshooting) ou `type: Decisão` (escolha técnica). Body com headings: `## Sintomas`, `## Causa`, `## Solução`, `## Relacionados`.
3. Atualizar `log.md` com entrada datada.
4. Atualizar `index.md` se documento novo foi criado.

### 4.6 Ao executar git commit

Antes de commitar, verificar se alterações no código afetam conceitos existentes no bundle `knowledge/`.

#### Procedimento

1. Analisar arquivos staged (`git diff --cached --name-only`) e identificar quais áreas do sistema foram alteradas (tabelas, endpoints, componentes, configs, etc.).
2. Consultar `knowledge/index.md` para localizar conceitos potencialmente impactados.
3. Se alteração **invalida ou desatualiza** informação documentada no bundle (schema mudou, endpoint renomeado, config alterada, comportamento diferente): atualizar conceitos afetados antes de commitar.
4. Se alteração **adiciona** funcionalidade nova ainda não documentada: propor criação de conceito (não bloquear commit).
5. Incluir atualizações do bundle no mesmo commit quando possível (código + docs = atômico).
6. Registrar em `log.md` se conceitos foram atualizados.

#### Quando NÃO atualizar bundle

- Refatoração interna sem mudança de comportamento externo
- Alterações em arquivos de teste apenas
- Mudanças cosméticas (formatação, lint fixes)
- Código WIP que será alterado novamente em breve

---

## 5. Convenções adicionais

- **Idioma**: seguir idioma do projeto (definido por steering de idioma).
- **Timestamps**: sempre UTC ou timezone do projeto (`America/Sao_Paulo`).
- **Nomes de arquivo**: kebab-case, sem acentos. Ex: `tabela-pedidos.md`, `api-usuarios.md`.
- **Tags**: lowercase, sem espaços. Usar pelo menos 3 níveis de especificidade:
  1. **Genérica** — domínio ou área ampla (ex: `backend`, `frontend`, `infra`, `seguranca`)
  2. **Intermediária** — feature ou subsistema (ex: `autenticacao`, `pedidos`, `api`, `streaming`)
  3. **Específica** — assunto concreto do documento (ex: `jwt-refresh`, `webhook-retry`, `sse-reconnect`)

  Exemplo completo para documento sobre estratégia de retry em webhooks de pagamento:
  ```yaml
  tags: [backend, integracoes, pagamentos, webhook-retry, resiliencia]
  #       ^genérica  ^intermediária  ^intermediária  ^específica  ^específica
  ```
- **Type values sugeridos para projetos de software**: `Tabela`, `API Endpoint`, `Componente`, `Composable`, `Middleware`, `Plugin`, `Configuração`, `Playbook`, `Conceito`, `Decisão`, `Referência`.
