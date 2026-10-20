# Primary Key e Foreign Key

## Primary Key
```sql
id_cliente BIGSERIAL PRIMARY KEY
```

Identifica uma linha de forma única.

## Foreign Key
```sql
id_cliente BIGINT NOT NULL
REFERENCES clientes(id_cliente)
```

Impede um pedido apontar para cliente inexistente.

Exemplo:

```text
cliente 10 existe → pedido com id_cliente 10 ✅
cliente 999 não existe → pedido com id_cliente 999 ❌
```
