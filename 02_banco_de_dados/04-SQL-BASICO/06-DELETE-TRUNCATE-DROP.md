# `DELETE`, `TRUNCATE` e `DROP`

## `DELETE`
```sql
DELETE FROM clientes
WHERE id_cliente = 2;
```

Perigoso:

```sql
DELETE FROM clientes;
```

## `TRUNCATE`
Remove todas as linhas rapidamente.

```sql
TRUNCATE TABLE clientes;
```

Reiniciando sequências:

```sql
TRUNCATE TABLE clientes RESTART IDENTITY;
```

## `DROP`
Remove a estrutura:

```sql
DROP TABLE clientes;
```

> [!DANGER]
> `DROP` e `TRUNCATE` exigem muita atenção em ambiente real.
