# Query Tool e segurança

Uma aba pode estar conectada em:

```text
movies/postgres@postgres
```

enquanto outra está em:

```text
loja/postgres@postgres
```

Clicar em `loja` na árvore não necessariamente altera uma aba já aberta.

## Atenção à seleção
Se você selecionar:

```sql
UPDATE clientes
SET telefone = '21999999999'
```

mas deixar o `WHERE` fora da seleção, todas as linhas podem ser alteradas.

Sempre confira a conexão no topo da aba.
