# `INSERT INTO`

## Um registro
```sql
INSERT INTO clientes (
    nome,
    sobrenome,
    email,
    telefone
)
VALUES (
    'Ana',
    'Silva',
    'ana@email.com',
    '21999990000'
);
```

## Vários registros
```sql
INSERT INTO produtos (nome, preco, estoque) VALUES
('Mouse', 59.90, 20),
('Teclado', 129.90, 10),
('Monitor', 899.00, 5);
```

Não precisamos informar `id_produto` se ele for `BIGSERIAL`, nem `criado_em` se houver `DEFAULT now()`.
