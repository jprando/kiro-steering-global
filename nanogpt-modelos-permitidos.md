---
inclusion: auto
description: Modelos permitidos no NanoGPT (MCP chat/vision e Batch API). Ativar ao usar nanogpt_chat, nanogpt_vision, ou montar jobs batch.
---

# NanoGPT — Modelos Permitidos

API Key: `sk-nano-e0f3cfe6-cd11-42ef-b348-e8158ac41f10`

Ao usar ferramentas do MCP server `nanogpt` (`nanogpt_chat`, `nanogpt_vision`, etc.) ou montar jobs via Batch API (ver steering `Global/nanogpt-batch.md`), usar **SOMENTE** os seguintes modelos:

- `deepseek/deepseek-v4-flash-0731:thinking`
- `minimax/minimax-m3:thinking`
- `deepseek/deepseek-v4-flash:thinking`
- `moonshotai/kimi-k2.6:thinking`
- `xiaomi/mimo-v2.5-pro:thinking`
- `zai-org/glm-5.2:thinking`
- `deepseek/deepseek-v4-pro-cheaper:thinking`
- `alibaba/qwen3.6-27b:thinking`
- `nvidia/nemotron-3-ultra-550b-a55b:thinking`
- `google/gemma-4-31b-it:thinking`
- `Qwen/Qwen3.6-35B-A3B:thinking`
- `google/gemma-4-26b-a4b-it:thinking`
- `deepseek/deepseek-v4-flash-latest`
- `minimax/minimax-latest`
- `moonshotai/kimi-latest`
- `zai-org/glm-latest`
- `deepseek/deepseek-latest`
- `deepseek/deepseek-v4-flash-0731`
- `minimax/minimax-m3`
- `zai-org/glm-5.2`
- `deepseek/deepseek-v4-flash`
- `moonshotai/kimi-k2.6`
- `alibaba/qwen3.6-27b`
- `Qwen/Qwen3.6-35B-A3B`
- `deepseek/deepseek-v4-pro`
- `moonshotai/kimi-k2.7-code`
- `xiaomi/mimo-v2.5-pro`
- `cohere/north-mini-code`
- `nvidia/nemotron-3-ultra-550b-a55b`
- `nvidia/nemotron-3-super-120b-a12b`
- `google/gemma-4-26b-a4b-it`
- `google/gemma-4-31b-it`
- `openai/gpt-oss-120b`
- `openai/gpt-oss-20b`
- `meta-llama/llama-4-maverick`
- `meta-llama/llama-4-scout`

## Regras

1. **Nunca** usar modelos da OpenAI (gpt-*), Anthropic (claude-*), ou Google (gemini-*) via NanoGPT.
2. Se nenhum modelo for especificado pelo usuário, usar `deepseek/deepseek-v4-flash-latest` como padrão.
3. Para tarefas de código, preferir `meta/muse-spark-1.2-contributor`.
4. Se o usuário pedir explicitamente um modelo fora da lista, informar que apenas os modelos acima estão permitidos e sugerir o mais adequado da lista.
5. sempre que possivel evitar os modelos que tem o sufixo `-cheaper`
