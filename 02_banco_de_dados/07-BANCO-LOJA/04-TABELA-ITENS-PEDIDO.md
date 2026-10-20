# Tabela `itens_pedido`

```sql
CREATE TABLE itens_pedido (
    id_pedido BIGINT NOT NULL
        REFERENCES pedidos(id_pedido)
        ON DELETE CASCADE,

    id_produto BIGINT NOT NULL
        REFERENCES produtos(id_produto),

    quantidade INTEGER NOT NULL
        CHECK (quantidade > 0),

    preco_unit NUMERIC(10,2) NOT NULL
        CHECK (preco_unit >= 0),

    PRIMARY KEY (id_pedido, id_produto)
);
```

Resolve:
```text
pedidos N ─── N produtos
```

por meio de:
```text
pedidos 1 ─── N itens_pedido N ─── 1 produtos
```
