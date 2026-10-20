# Tabela `pedidos`

```sql
CREATE TABLE pedidos (
    id_pedido BIGSERIAL PRIMARY KEY,

    id_cliente BIGINT NOT NULL
        REFERENCES clientes(id_cliente),

    status VARCHAR(20) NOT NULL DEFAULT 'aberto'
        CHECK (
            status IN (
                'aberto',
                'pago',
                'enviado',
                'cancelado'
            )
        ),

    criado_em TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## Relação
```text
clientes 1 ─── N pedidos
```

A FK impede pedido para cliente inexistente.
