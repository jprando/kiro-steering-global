---
inclusion: auto
description: AI Batch API — jobs assíncronos em massa via JSONL + polling. Ativar ao processar 5+ chat completions em paralelo.
---

# NanoGPT Batch API — Diretrizes de Uso

## Visao Geral

A Batch API do NanoGPT permite executar jobs assincronos de chat completion em massa via `/v1/chat/completions`. Ideal para tarefas offline onde latencia nao importa: classificacao, sumarizacao, evals, geracao de dados sinteticos, processamento de documentos, e analise de codigo.

## Base URL

```
https://api.nano-gpt.com/api/v1
```

Autenticacao: `Authorization: Bearer sk-nano-e0f3cfe6-cd11-42ef-b348-e8158ac41f10`

API Key: `sk-nano-e0f3cfe6-cd11-42ef-b348-e8158ac41f10`

IMPORTANTE: Usar `api.nano-gpt.com` (nao `nano-gpt.com`) para uploads de arquivos batch.

## Modelos Open-Source Suportados no Batch

Os seguintes modelos open-source estao disponiveis na Batch API:

- `deepseek/deepseek-v4-flash`
- `deepseek/deepseek-v4-pro`
- `zai-org/glm-5.1`
- `zai-org/glm-5.2`
- `moonshotai/kimi-k2.5`
- `moonshotai/kimi-k2.6`
- `moonshotai/kimi-k2.7-code`
- `minimax/minimax-m2.7`
- `minimax/minimax-m3`
- `nvidia/nemotron-3-ultra-nvfp4`
- `qwen/qwen3.6-plus`
- `qwen/qwen3.7-plus`
- `gpt-oss-120b`
- `gpt-oss-20b`

NOTA: Esta lista evolui rapidamente. Os IDs exatos podem mudar. Se um ID falhar, tentar variantes (ex: com/sem prefixo de provider, com/sem versao). Em caso de duvida, usar a API sincrona via MCP `nanogpt_chat` para confirmar o ID do modelo.

## Modelos Preferidos (restritos pela steering nanogpt-modelos-permitidos)

Ao criar batch jobs, usar preferencialmente:

1. `zai-org/glm-latest` — **modelo padrao** para todas as tarefas
2. `moonshotai/kimi-k2.7-code` — para tarefas especificas de codigo (quando solicitado)
3. `deepseek/deepseek-latest` — alternativa para tarefas gerais
4. `minimax/minimax-latest` — alternativa

Se nenhum modelo for especificado pelo usuario, usar `zai-org/glm-latest`.

## Fluxo de Trabalho

### 1. Montar o arquivo JSONL

Cada linha eh um JSON object com:

```jsonl
{"custom_id":"<id-unico>","method":"POST","url":"/v1/chat/completions","body":{"model":"<modelo>","messages":[...],"max_tokens":<N>}}
```

Regras obrigatorias:
- `custom_id`: unico e nao-vazio em cada linha
- `method`: sempre `"POST"`
- `url`: sempre `"/v1/chat/completions"`
- `body.model`: obrigatorio
- `body.messages`: obrigatorio
- `body.max_tokens` ou `body.max_completion_tokens`: obrigatorio
- Todas as linhas devem usar o MESMO modelo
- Nao misturar familias de modelo no mesmo arquivo
- `stream: true` NAO eh suportado

### 2. Upload do arquivo

```bash
curl https://api.nano-gpt.com/api/v1/files \
  -H "Authorization: Bearer sk-nano-e0f3cfe6-cd11-42ef-b348-e8158ac41f10" \
  -F purpose=batch \
  -F file=@batch.jsonl
```

### 3. Criar o batch

```bash
curl https://api.nano-gpt.com/api/v1/batches \
  -H "Authorization: Bearer sk-nano-e0f3cfe6-cd11-42ef-b348-e8158ac41f10" \
  -H "Content-Type: application/json" \
  -d '{"input_file_id":"<file_id>","endpoint":"/v1/chat/completions","completion_window":"24h"}'
```

### 4. Polling

```bash
curl https://api.nano-gpt.com/api/v1/batches/<batch_id> \
  -H "Authorization: Bearer sk-nano-e0f3cfe6-cd11-42ef-b348-e8158ac41f10"
```

Status possiveis: `validating`, `in_progress`, `finalizing`, `completed`, `failed`, `expired`, `cancelling`, `cancelled`.

### 5. Download do resultado

Quando status for `completed`, o campo `output_file_id` estara presente:

```bash
curl https://api.nano-gpt.com/api/v1/files/<output_file_id>/content \
  -H "Authorization: Bearer sk-nano-e0f3cfe6-cd11-42ef-b348-e8158ac41f10"
```

## Casos de Uso no Desenvolvimento

- **Code review em massa**: enviar N classes para revisao automatizada
- **Geracao de Javadoc/JSDoc**: documentar metodos publicos em batch
- **Geracao de dados sinteticos**: fixtures, mocks, dados de teste
- **Classificacao de codigo**: categorizar complexidade, detectar padroes
- **Sumarizacao de documentos**: processar specs, requirements, RFCs
- **Analise de imagens**: screenshots de UI, wireframes, diagramas

## Billing

- Batch jobs usam saldo da conta NanoGPT
- Batch tem desconto vs. API sincrona
- Cobranca final calculada apos o job completar (baseada em uso real)
- Requests que falham nao sao cobrados

## Limites e Restricoes

- Somente `/v1/chat/completions` (nao suporta embeddings, responses, etc.)
- Tools, functions, response_format, audio NAO suportados nesta versao
- Imagens suportadas via `image_url` (http, https, ou base64 data URI)
- Tipos de imagem: PNG, JPEG, GIF, WebP

## Quando Usar Batch vs. MCP Sincrono

| Cenario | Usar |
|---|---|
| 1-5 requisicoes, resposta imediata necessaria | MCP `nanogpt_chat` |
| 10+ requisicoes, pode esperar | Batch API |
| Precisa de tools/functions | MCP `nanogpt_chat` (batch nao suporta) |
| Processamento de imagens em massa | Batch API |
| Interacao iterativa (refinar resposta) | MCP `nanogpt_chat` |

## Padrao de Orquestracao: Kiro + Batch API

O Kiro atua como **orquestrador** e a Batch API como **workers** para tarefas paralelizaveis.

### Arquitetura

```
KIRO (Orquestrador)
  1. Entende a tarefa do usuario
  2. Le os arquivos relevantes do projeto
  3. Monta o JSONL com N requisicoes (1 por unidade de trabalho)
  4. Upload + cria batch
  5. Faz polling ate completar
  6. Baixa resultados
  7. Processa e aplica os resultados no projeto (edita arquivos, gera relatorios, etc.)

NanoGPT Batch API (Workers)
  - Processa N requisicoes em paralelo no server
  - Cada request = 1 worker independente
  - Modelo open-source (GLM, DeepSeek, MiniMax, Kimi)
  - Retorna tudo de uma vez no output JSONL
```

### Quando usar este padrao

| Caso de Uso | Kiro faz | Workers fazem |
|---|---|---|
| Code review em massa | Le N classes, monta batch | Cada worker revisa 1 classe |
| Geracao de testes | Identifica metodos sem teste | Cada worker gera teste para 1 metodo |
| Geracao de Javadoc/JSDoc | Lista classes sem doc | Cada worker documenta 1 classe |
| Migracao/refactoring | Analisa estrutura, define regras | Cada worker migra 1 arquivo |
| i18n / traducao | Extrai strings | Cada worker traduz 1 bloco |
| Analise de dependencias | Lista modulos | Cada worker analisa 1 modulo |

### Vantagens vs. MCP sincrono (N chamadas sequenciais)

| Aspecto | MCP sincrono (N chamadas) | Batch API |
|---|---|---|
| Velocidade | Sequencial (1 por vez) | Paralelo no server |
| Custo | Preco cheio | Com desconto |
| Contexto do Kiro | Gasta context window com N respostas | Processa offline, traz so resultado final |
| Resiliencia | Se a sessao cair, perde progresso | Batch continua rodando independente |

### Limitacoes do padrao

1. **Latencia minima**: batch nao eh instantaneo — pode levar segundos a minutos dependendo do volume
2. **Sem iteracao**: cada worker eh independente — nao da pra refinar resposta dentro do batch
3. **Sem tools/functions**: worker so faz chat completion puro
4. **Contexto limitado**: cada request precisa caber no context window do modelo
5. **Sessao do Kiro**: se a sessao compactar durante o polling, re-confirmar o batch_id pelos logs

### Fluxo de decisao

```
Tarefa recebida
  |
  ├─ Sao menos de 5 unidades de trabalho?
  |    └─ SIM → Usar MCP nanogpt_chat sequencial
  |
  ├─ Precisa de iteracao/refinamento?
  |    └─ SIM → Usar MCP nanogpt_chat
  |
  ├─ Precisa de tools/functions?
  |    └─ SIM → Usar MCP nanogpt_chat
  |
  └─ Sao 5+ unidades independentes, sem iteracao?
       └─ SIM → Usar Batch API (Kiro orquestra, batch executa)
```

### Configuracao de proxy

Todas as chamadas curl para a Batch API devem usar o proxy corporativo:

```bash
curl --proxy http://proxy.el.com.br:3128 ...
```

### Dica sobre max_tokens

Modelos com thinking (como `zai-org/glm-latest` que resolve para `glm-5.2:thinking`) gastam tokens no raciocinio interno. Para esses modelos, usar `max_tokens` maior (768-2048) para garantir que a resposta final nao seja cortada.
