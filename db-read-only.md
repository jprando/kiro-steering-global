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
