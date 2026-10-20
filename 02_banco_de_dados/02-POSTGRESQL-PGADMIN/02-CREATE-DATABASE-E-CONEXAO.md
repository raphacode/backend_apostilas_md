# Criar e selecionar o database

Abra o Query Tool conectado a um database existente, normalmente `postgres`.

```sql
CREATE DATABASE loja;
```

Atualize `Databases`.

## PostgreSQL não usa `USE loja`

No `psql`:
```text
\c loja
```

No pgAdmin, conecte a aba do Query Tool ao database `loja`.

## Confirmar o banco atual
```sql
SELECT current_database();
```

Isso **consulta**, não troca a conexão.
