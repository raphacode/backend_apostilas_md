# Tabela associativa para `N:N`

```sql
CREATE TABLE itens_pedido (
    id_pedido BIGINT NOT NULL
        REFERENCES pedidos(id_pedido)
        ON DELETE CASCADE,

    id_produto BIGINT NOT NULL
        REFERENCES produtos(id_produto),

    quantidade INTEGER NOT NULL CHECK (quantidade > 0),

    preco_unit NUMERIC(10,2) NOT NULL CHECK (preco_unit >= 0),

    PRIMARY KEY (id_pedido, id_produto)
);
```

## PK composta
```sql
PRIMARY KEY (id_pedido, id_produto)
```

## `ON DELETE CASCADE`
Se o pedido for apagado, seus itens também são apagados.

## `preco_unit`
Preserva o preço praticado na data da venda, mesmo que o preço atual do produto mude.
