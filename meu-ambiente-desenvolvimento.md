---
inclusion: always
description: Stack local — CachyOS, Fish, Node via Vite+/vpx, pnpm, uv, Docker. Ativar ao executar comandos ou configurar ambiente.
---

# Meu Ambiente de Desenvolvimento

## Sistema Operacional

- **CachyOS** (Arch-based, rolling release)
- Kernel: `7.1.x-cachyos` (PREEMPT_DYNAMIC)
- Arquitetura: x86_64
- Locale: `pt_BR.UTF-8` (todas as categorias)
- Timezone: `America/Sao_Paulo`

## Shell

- **Fish** (`/bin/fish`) — NÃO é bash/zsh
- A sintaxe de variáveis de ambiente é `set -gx VAR valor`, não `export VAR=valor`
- Configuração: `~/.config/fish/config.fish`
- Fastfetch desabilitado em terminais de IDE (VS Code, Zed, Kiro)
- CachyOS fish config carregada apenas fora de IDEs

## Gerenciamento de Node.js

- **Vite+** (`vp`) — gerencia Node.js, pnpm e ferramentas do ecossistema
- Binários em `~/.vite-plus/bin/` (node, pnpm, vpx)
- Node.js: `v24.x` (versão atual, pode mudar via `vp env use`)
- pnpm: `11.x` (gerenciado pelo Vite+)
- `vpx` — substituto de `npx`, está em `~/.vite-plus/bin/vpx`

## Gerenciadores de Pacotes

| Ferramenta | Versão | Path |
|---|---|---|
| uvx | — | `/usr/bin/uvx` |
| pnpm | 11.x | `~/.vite-plus/bin/pnpm` |
| bun | 1.4.x | `~/.bun/bin/bun` |
| uv (Python) | 0.11.x | `/usr/bin/uv` |

## Runtimes e Linguagens

| Runtime | Versão | Path |
|---|---|---|
| Node.js | v24.x | `~/.vite-plus/bin/node` |
| Python | 3.14.x | `/usr/bin/python3` |
| Bun | 1.4.x | `~/.bun/bin/bun` |

## Ferramentas de Desenvolvimento

| Ferramenta | Versão | Path |
|---|---|---|
| Git | 2.54.x | `/usr/bin/git` |
| Docker | 29.x | — |
| Docker Compose | 5.x | — |
| rtk | — | `~/.local/bin/rtk` |
| vpx | — | `~/.vite-plus/bin/vpx` |

## Git

- Usuário: Jeudi Prando (`jeudiprando@gmail.com`)
- Branch padrão: `main`
- Editor: não configurado (padrão do sistema)
- **GitHub CLI (`gh`) NÃO está instalada** — não usar comandos `gh`

## Particularidades Importantes

### 1. Node.js via Vite+ (NÃO via nvm/fnm/volta)

O Node.js **não** é gerenciado por nvm, fnm ou volta. É gerenciado pelo **Vite+** (`vp`).
Não assumir que `nvm use`, `fnm use` ou `volta pin` funcionam.

### 2. Shell é Fish (NÃO bash/zsh)

- Não usar sintaxe bash para exportar variáveis (`export X=y`)
- Não usar `source ~/.bashrc` ou `~/.zshrc`
- Arrays: `set -l arr a b c` (não `arr=(a b c)`)
- Condicionais: `if ... end` (não `if ... fi`)
- O Kiro executa comandos via bash internamente, mas o ambiente do usuário é Fish

### 3. vpx em vez de npx

Usar `vpx` para executar binários de pacotes. O `npx` pode não estar disponível ou se comportar diferente.

### 4. pnpm como gerenciador exclusivo (JavaScript/TypeScript)

Nunca usar `npm` ou `yarn`. Sempre `pnpm`.

### 5. Arch-based (pacman)

Pacotes do sistema são instalados via `pacman` ou `yay`/`paru` (AUR helpers), não `apt`, `dnf` ou `brew`.

### 6. Sem GitHub CLI

O `gh` não está instalado. Para operações com GitHub, usar a interface web ou Git diretamente.

### 7. Python via uv

Para ferramentas Python, preferir `uv` e `uvx` em vez de `pip` global.

### 8. Playwright CLI — Sempre modo headed (NÃO headless)

O `playwright-cli` deve ser executado **SEMPRE** no modo **headed** (`--headed`), para que o navegador fique visível e eu possa acompanhar visualmente o que está sendo feito.

```bash
# ✅ Correto — navegador visível
/usr/bin/playwright-cli open --headed http://localhost:8080

# ❌ Errado — headless (invisível)
/usr/bin/playwright-cli open http://localhost:8080
```

Nunca usar `--headless` ou omitir `--headed`. O objetivo é sempre ver a execução no navegador.

### 9. Timezone: America/Sao_Paulo

O timezone padrão para todos os projetos é `America/Sao_Paulo`. Onde for possível definir timezone — Docker Compose (`TZ`), `.env`, configurações de runtime, CI/CD — sempre usar `America/Sao_Paulo`.

Exemplos:
- Docker Compose: `TZ: America/Sao_Paulo` no `environment` dos services
- `.env`: `TZ=America/Sao_Paulo`
- Node.js: `process.env.TZ` (definido via ambiente, nunca hardcoded no código)

**PostgreSQL e `PGTZ`:**
- `TZ` no container do PostgreSQL é suficiente — o servidor herda o timezone do sistema para `now()`, `CURRENT_TIMESTAMP` e conversões de `timestamptz`.
- `PGTZ` é variável do **cliente** (libpq), não do servidor. Define o timezone da sessão de quem conecta (equivalente a `SET timezone = '...'`).
- Não é necessário definir `PGTZ` no container do postgres. No container da aplicação (cliente), `TZ` já definido faz o libpq herdar o timezone correto sem precisar de `PGTZ` explícito.
- Resumo: apenas `TZ` em todos os services é suficiente. Não duplicar com `PGTZ`.

### 10. Sem código inline no shell

**EVITE SEMPRE QUE PUDER** executar código inline no terminal via flags como `-c`, `-e`, `--eval`, etc. Sempre criar um arquivo de script separado e executá-lo.

```bash
# ❌ Errado — código inline
node -e "console.log(JSON.parse(require('fs').readFileSync('x.json')))"
python3 -c "import json; print(json.load(open('x.json')))"

# ✅ Correto — script em arquivo separado
rtk node script.js
rtk python3 script.py
```

Isso vale para qualquer runtime (node, python, ruby, perl, etc.). Se precisa executar lógica, colocar em arquivo.

### 11. Git commit multilinha — usar arquivo de mensagem

O terminal do Kiro (bash interno) **não suporta** mensagens de commit com múltiplas linhas passadas diretamente via `-m`. Aspas simples e duplas com quebras de linha quebram o parsing do shell.

**Solução**: Sempre usar arquivo temporário com `git commit -F`:

```bash
# ✅ Correto — mensagem multilinha via arquivo
# 1. Criar arquivo com a mensagem (via fs_write)
# 2. Executar commit referenciando o arquivo
rtk git commit -F .git/COMMIT_MSG_TMP
# 3. Arquivo pode ser sobrescrito nas próximas vezes (não precisa deletar)

# ❌ Errado — quebra no shell do Kiro
rtk git commit -m "subject

body com múltiplas linhas"
```

Isso se aplica a **qualquer** mensagem com body (mais de uma linha). Subject-only (`-m "uma linha"`) funciona normalmente.

### 12. Pasta `.local` para arquivos temporários

Se o workspace/projeto aberto no Kiro possuir uma pasta `.local` na raiz, **usar essa pasta para criar arquivos temporários** (scripts auxiliares, dumps, saídas de debug, etc.).

Convenção: a pasta `.local` geralmente já está registrada no `.gitignore` do projeto.

**Procedimento:**
1. Verificar se `.local/` existe na raiz do projeto.
2. Se existir, usar `.local/` como destino de arquivos temporários.
3. Antes de criar arquivos, verificar se `.local` está no `.gitignore`.
4. Se `.local` **não** estiver no `.gitignore`: **perguntar ao usuário** se pode adicionar `.local` ao `.gitignore` antes de prosseguir. Nunca alterar o `.gitignore` sem confirmação explícita.

Se `.local/` **não** existir no projeto, criar temporários onde fizer mais sentido (raiz, `/tmp`, etc.) — sem restrição.

```bash
# ✅ Correto — .local/ existe, usar como destino
rtk node .local/script-temp.js
rtk python3 .local/verifica-algo.py

# ❌ Errado — .local/ existe mas temporário criado fora dela
node /tmp/script.js
node script-temp.js
```
