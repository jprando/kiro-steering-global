---
inclusion: always
description: "Força modo read-only em todas conexões ao banco. Proíbe execução direta de INSERT/UPDATE/DELETE/DROP/ALTER/CREATE pelo agent."
---

# Banco de Dados — Modo Read-Only Obrigatório

## Regra absoluta

**TODA** conexão ao banco de dados feita pelo code agent DEVE ser em modo **READ-ONLY**. Sem exceção.

Isso significa:
- Queries `SELECT` — permitidas livremente.
- Queries de escrita (`INSERT`, `UPDATE`, `DELETE`, `DROP`, `TRUNCATE`, `ALTER`, `CREATE`) — **PERMANENTEMENTE PROIBIDAS** para execução direta pelo code agent.

## Proibições explícitas

O code agent está **PERMANENTEMENTE PROIBIDO** de executar os seguintes comandos no banco de dados:

- `DELETE`
- `UPDATE`
- `INSERT`
- `DROP`
- `TRUNCATE`
- `ALTER`
- `CREATE`

Nenhuma justificativa, contexto ou necessidade técnica autoriza o code agent a executar comandos de escrita ou destrutivos no banco. **NUNCA.**

## Se precisar de escrita

Quando a tarefa exigir escrita no banco para ser concluída:

1. **PARAR** a execução.
2. **EXIBIR** para o usuário desenvolvedor:
   - O comando SQL exato que precisa ser executado.
   - Explicação do que o comando faz.
   - Qual tabela/dados serão afetados.
   - Se é reversível ou não.
3. **AGUARDAR** confirmação explícita do usuário para prosseguir (e mesmo assim, o usuário é quem executa o comando manualmente).

O code agent **NUNCA** executa o comando de escrita por conta própria — apenas apresenta e orienta.

## Conexão

Ao conectar no banco (via `psql`, driver, ORM, ou qualquer outro meio):

- Usar `default_transaction_read_only = on` quando possível.
- Usar `SET SESSION CHARACTERISTICS AS TRANSACTION READ ONLY;` como primeiro comando da sessão.
- Se o driver/ferramenta suportar flag de read-only, ativá-la.

## psql — FORÇAR conexão somente leitura (OBRIGATÓRIO)

**TODO** uso do `psql` pelo code agent DEVE forçar a sessão como somente leitura. Apenas `SELECT`/leitura é permitido. Sem isso a conexão NÃO é aceitável.

### 1. Variável de ambiente (preferencial e sempre)

Exportar antes de qualquer execução do `psql`:

```bash
export PGOPTIONS="-c default_transaction_read_only=on"
```

Exemplo de uso:

```bash
export PGOPTIONS="-c default_transaction_read_only=on"
PGPASSWORD="***" psql -h "$HOST" -p "$PORT" -U "$USER" -d "$DB" -c "SELECT ..."
```

Essa variável aplica `default_transaction_read_only=on` a **toda sessão psql** e faz o PostgreSQL recusar qualquer comando de escrita (INSERT/UPDATE/DELETE/DROP/TRUNCATE/ALTER/CREATE) com erro.

### 2. Flag nativa do psql

Usar quando a versão/situação permitir:

```bash
psql --set=default_transaction_read_only=on -c "SELECT ..."
```

### 3. SQL de sessão — pré-comando obrigatório

Se as opções acima não forem aplicáveis ou como defesa em profundidade, executar **antes** do SQL que pretende executar:

```sql
SET SESSION CHARACTERISTICS AS TRANSACTION READ ONLY;
SELECT ...;
```

## Erros por conexão somente leitura — COMPORTAMENTO ESPERADO E IDEAL

Se o SQL executado falhar porque a conexão está em modo somente leitura, **isso é esperado e ideal** — significa que a proteção está funcionando.

- **NUNCA** desarmar a conexão read-only para "resolver" o erro.
- **NUNCA** contornar o erro reonectando sem `PGOPTIONS`/read-only ou com outro driver gravável.
- Quando ocorrer erro porque o SQL a ser executado **não é somente leitura** (tentativa de INSERT/UPDATE/DELETE/DROP/TRUNCATE/ALTER/CREATE): **PARAR** — não tentar resolver ou executar a query de outro jeito.
- **EXPLICAR** ao usuário o que aconteceu (query de escrita bloqueada pela conexão read-only) e **solicitar que o usuário execute a consulta manualmente**, conforme a seção [Se precisar de escrita](#se-precisar-de-escrita).

## Resumo

| Operação | Permitida? |
|----------|-----------|
| SELECT / leitura | Sim |
| INSERT | Não — exibir para usuário executar |
| UPDATE | Não — exibir para usuário executar |
| DELETE | Não — exibir para usuário executar |
| DROP | Não — exibir para usuário executar |
| TRUNCATE | Não — exibir para usuário executar |
| ALTER | Não — exibir para usuário executar |
| CREATE | Não — exibir para usuário executar |

## Checklist obrigatório — antes de QUALQUER psql

1. `export PGOPTIONS="-c default_transaction_read_only=on"` **sempre** exportado na sessão.
2. Nunca executar INSERT/UPDATE/DELETE/DROP/TRUNCATE/ALTER/CREATE — erro de read-only é a proteção funcionando (esperado e ideal).
3. Ícone mental: **toda** sessão psql é somente leitura, sem exceções.
