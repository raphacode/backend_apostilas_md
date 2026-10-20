# DDL e Constraints

## DDL
Comandos de estrutura:

```sql
CREATE DATABASE
CREATE TABLE
ALTER TABLE
DROP TABLE
TRUNCATE TABLE
```

## Constraints

### `NOT NULL`
```sql
nome VARCHAR(255) NOT NULL
```

### `UNIQUE`
```sql
email VARCHAR(255) UNIQUE
```

Permite impedir duplicidade. Em PostgreSQL, múltiplos `NULL` podem existir.

### `CHECK`
```sql
estoque INTEGER CHECK (estoque >= 0)
```

### `DEFAULT`
```sql
ativo BOOLEAN NOT NULL DEFAULT true
```

### `PRIMARY KEY`
```sql
id_produto BIGSERIAL PRIMARY KEY
```

### `REFERENCES`
```sql
id_cliente BIGINT REFERENCES clientes(id_cliente)
```
