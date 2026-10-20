# `ORDER BY`, `LIMIT` e `OFFSET`

## Ordenação
```sql
SELECT *
FROM clientes
ORDER BY id_cliente ASC;
```

```sql
ORDER BY id_cliente DESC;
```

## `LIMIT`
```sql
LIMIT 10
```

Mostra no máximo 10 registros.

## `OFFSET`
```sql
OFFSET 20
```

Pula 20 antes de começar a mostrar.

## Paginação

Página 1:
```sql
LIMIT 10 OFFSET 0;
```

Página 2:
```sql
LIMIT 10 OFFSET 10;
```

Página 3:
```sql
LIMIT 10 OFFSET 20;
```

Fórmula:

```text
OFFSET = (pagina - 1) × tamanho_da_pagina
```
