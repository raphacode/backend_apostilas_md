# Transações

## `BEGIN`
```sql
BEGIN;
```

Inicia a transação.

## `COMMIT`
```sql
COMMIT;
```

Confirma definitivamente.

## `ROLLBACK`
```sql
ROLLBACK;
```

Desfaz alterações ainda não confirmadas.

## Exemplo
```sql
BEGIN;

UPDATE clientes
SET telefone = '21999999999'
WHERE id_cliente = 1
RETURNING *;

-- correto:
COMMIT;

-- errado:
-- ROLLBACK;
```

Depois do `COMMIT`, o `ROLLBACK` não desfaz aquela transação.
