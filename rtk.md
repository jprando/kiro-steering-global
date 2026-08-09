---
inclusion: always
description: Proxy CLI obrigatório (rtk) — prefixar TODOS comandos de terminal. Filtra output economizando 60-99% tokens.
---
# RTK — Rust Token Killer

**Usage**: Proxy CLI que filtra output de comandos antes de entregar ao LLM, economizando 60-99% de tokens.

## Regra principal

**SEMPRE** prefixe com `rtk` (ou $HOME/.local/bin/rtk) **TODOS** os comandos executados no terminal — sem exceção.
Não importa se o comando está na lista abaixo ou não. Todo e qualquer comando bash executado via ferramenta de terminal DEVE ser prefixado com `rtk`.
Isso é transparente — o RTK executa o comando real e filtra a saída automaticamente. Para comandos não listados, o RTK faz passthrough (executa sem filtrar), mas ainda assim DEVE ser usado.

> ⚠️ **NUNCA** execute um comando no terminal sem o prefixo `rtk`. Se precisar rodar `find` 10 vezes, use `rtk find` 10 vezes. Sem exceções — exceto os comandos listados na seção "Exceções" abaixo.

## Exceções — NÃO prefixar com `rtk`

### 1. Comandos interativos

Comandos interativos (que mantêm um processo aberto esperando input do usuário) **NÃO devem** ser prefixados com `rtk`, pois o RTK não suporta I/O interativo.

```bash
# ✅ Executar diretamente, sem rtk
/usr/bin/playwright-cli open --headed http://localhost:3000
vim arquivo.txt
nano arquivo.txt
less arquivo.txt
htop
```

### 2. Comandos com scripts inline multilinha

Comandos que recebem código inline via flag (`-c`, `-e`, `--eval`, etc.) com **múltiplas linhas ou lógica complexa** (imports, loops, blocos with/try) **NÃO devem** ser prefixados com `rtk`, pois o parsing de argumentos do RTK corrompe o script.

```bash
# ❌ NÃO funciona com rtk — script multilinha complexo
rtk python3 -c "
import json
with open('file.json') as f:
    data = json.load(f)
print(data)
"

# ✅ Executar diretamente, sem rtk
python3 -c "
import json
with open('file.json') as f:
    data = json.load(f)
print(data)
"

# ✅ Alternativa: usar rtk com script em arquivo separado
rtk python3 script.py

# ✅ One-liners simples PODEM usar rtk normalmente
rtk python3 -c "print('hello')"
rtk node -e "console.log(1)"
```

**Regra prática:** se o argumento de `-c`/`-e` contém quebras de linha, imports, ou blocos de código (with, for, if, try), execute **sem** `rtk`.

### 3. `ls`

O comando `ls` **NÃO deve** ser prefixado com `rtk`. Execute diretamente.

```bash
# ✅ Executar diretamente, sem rtk
ls
ls -la src/
```

### Regra geral para exceções

Se o comando é **sabidamente interativo** (abre uma UI, espera input contínuo do usuário, ou mantém um processo de longa duração com interação), usa **scripts inline multilinha**, ou é `ls`, execute-o **sem** o prefixo `rtk`.

## Meta Commands (usar rtk diretamente, sem comando aninhado)

```bash
rtk gain              # Mostrar analytics de economia de tokens
rtk gain --history    # Histórico de uso com savings
rtk discover          # Analisar oportunidades perdidas
rtk proxy <cmd>       # Executar raw (sem filtering, para debug)
rtk smart <file>      # Resumo heurístico de código (2 linhas, assinaturas)
rtk --version         # Verificar instalação
```

## Global flags (aplicáveis a qualquer comando rtk)

| Flag | Efeito |
|------|--------|
| `--ultra-compact` | Formato inline com ícones ASCII — redução extra |
| `-v` / `-vv` / `-vvv` | Mostra detalhes de filtragem no stderr |

⚠️ Não usar `-u` com git — conflita com `--set-upstream`. Usar `--ultra-compact` por extenso.

## Comandos cobertos — sempre prefixar com `rtk`

### Git

```bash
rtk git status
rtk git log
rtk git diff
rtk git show
rtk git stash list
```

### GitHub CLI

```bash
rtk gh pr view
rtk gh pr checks
rtk gh run list
rtk gh issue view
```

### Graphite (Stacked PRs)

```bash
rtk gt log
rtk gt status
```

### Cargo / Rust

```bash
rtk cargo test
rtk cargo nextest
rtk cargo build
rtk cargo check
rtk cargo clippy
```

### JavaScript / TypeScript

```bash
rtk jest
rtk vitest
rtk tsc
rtk eslint
rtk pnpm list
rtk pnpm outdated
rtk next build
rtk prisma migrate
rtk playwright test
```

### Python

```bash
rtk pytest
rtk ruff check
rtk mypy
rtk pip install
```

### Go

```bash
rtk go test
rtk golangci-lint run
rtk go build
```

### Ruby

```bash
rtk rspec
rtk rubocop
rtk rake
```

### .NET

```bash
rtk dotnet build
rtk dotnet test
rtk dotnet format
```

### Docker / Kubernetes

```bash
rtk docker ps
rtk docker images
rtk docker logs
rtk docker compose up
rtk kubectl get pods
rtk kubectl logs
```

### Arquivos e busca

```bash
rtk find
rtk grep
rtk diff
rtk wc
rtk cat <file>
rtk head <file>
rtk tail <file>
```

### Cloud e dados

```bash
rtk aws
rtk psql
rtk curl
```

## Comandos NÃO cobertos (pelo filtering)

Mesmo comandos que não estão na lista acima **DEVEM** ser prefixados com `rtk`.
O RTK faz passthrough (executa sem filtrar), mas o prefixo é obrigatório para rastreamento e consistência.

```bash
# Exemplo: comando não listado — ainda assim usar rtk
rtk which something
rtk echo "hello"
rtk wc -l arquivo.txt
```

> ⚠️ Lembre-se: scripts inline multilinha são exceção (ver seção "Exceções" acima).

## Verificação de instalação

```bash
rtk --version         # Deve exibir: rtk X.Y.Z
which rtk             # Verificar binário correto
```

⚠️ **Name collision**: Se `rtk gain` falhar, pode ser o pacote reachingforthejack/rtk (Rust Type Kit) instalado no lugar.
