# Cola SQL

## Estrutura
```sql
CREATE DATABASE
CREATE TABLE
ALTER TABLE
DROP TABLE
TRUNCATE TABLE
```

## CRUD
```sql
INSERT INTO
SELECT
UPDATE
DELETE
```

## Filtros
```sql
WHERE
ILIKE
BETWEEN
IN
IS NULL
```

## Ordenação/paginação
```sql
ORDER BY
LIMIT
OFFSET
```

## Relacionamentos
```sql
PRIMARY KEY
FOREIGN KEY
REFERENCES
INNER JOIN
LEFT JOIN
RIGHT JOIN
ON
```

## Agregações
```sql
COUNT
SUM
AVG
MIN
MAX
GROUP BY
HAVING
```

## Segurança/integridade
```sql
NOT NULL
UNIQUE
CHECK
DEFAULT
ON DELETE CASCADE
```

## Transação
```sql
BEGIN
COMMIT
ROLLBACK
```

## PostgreSQL
```sql
current_database()
generate_series()
LPAD()
CAST()
::TEXT
%
CROSS JOIN
```

## Supabase
```sql
SELECT current_database();
ALTER TABLE ... ENABLE ROW LEVEL SECURITY;
SELECT tablename, rowsecurity FROM pg_tables WHERE schemaname = 'public';
```
