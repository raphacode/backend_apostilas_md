# `SELECT` e filtros

## Todos
```sql
SELECT * FROM clientes;
```

## Específico
```sql
SELECT *
FROM clientes
WHERE id_cliente = 1;
```

## `IS NULL`
```sql
SELECT *
FROM clientes
WHERE email IS NULL;
```

## `ILIKE`
```sql
SELECT *
FROM clientes
WHERE nome ILIKE 'a%';
```

```sql
SELECT *
FROM clientes
WHERE nome ILIKE '%r%';
```

## `BETWEEN`
```sql
SELECT *
FROM produtos
WHERE preco BETWEEN 100 AND 500;
```

## `IN`
```sql
SELECT *
FROM produtos
WHERE id_produto IN (1,3,5);
```
