# Popular a Padaria

```sql
BEGIN;

INSERT INTO clientes (
    nome,
    sobrenome,
    email,
    telefone
) VALUES
('Ana', 'Silva', 'ana@padaria.com', '21990000001'),
('Bruno', 'Lima', 'bruno@padaria.com', '21990000002'),
('Carla', 'Souza', 'carla@padaria.com', '21990000003'),
('Diego', 'Santos', 'diego@padaria.com', '21990000004'),
('Elisa', 'Costa', 'elisa@padaria.com', '21990000005');

INSERT INTO categorias (nome) VALUES
('Paes'),
('Bolos'),
('Salgados'),
('Bebidas'),
('Doces');

INSERT INTO produtos (
    id_categoria,
    nome,
    preco,
    estoque
) VALUES
(1, 'Pao Frances', 1.20, 500),
(1, 'Pao de Queijo', 4.50, 120),
(1, 'Pao Integral', 12.90, 40),
(2, 'Bolo de Chocolate', 45.00, 10),
(2, 'Bolo de Cenoura', 39.90, 8),
(3, 'Coxinha', 8.50, 60),
(3, 'Empada', 9.00, 50),
(4, 'Cafe', 5.00, 100),
(4, 'Suco de Laranja', 8.00, 40),
(5, 'Brigadeiro', 3.50, 150);

INSERT INTO fornecedores (
    razao_social,
    cnpj,
    telefone
) VALUES
('Distribuidora Trigo Bom', '11.111.111/0001-11', '2122221111'),
('Laticinios Serra', '22.222.222/0001-22', '2122222222'),
('Bebidas Vale', '33.333.333/0001-33', '2122223333');

INSERT INTO produto_fornecedor (
    id_produto,
    id_fornecedor,
    preco_compra,
    prazo_dias
) VALUES
(1, 1, 0.55, 1),
(2, 1, 2.20, 1),
(2, 2, 2.05, 2),
(4, 1, 20.00, 2),
(4, 2, 18.50, 3),
(8, 3, 2.00, 1),
(9, 3, 3.50, 1);

INSERT INTO pedidos (
    id_cliente,
    status
) VALUES
(1, 'pago'),
(1, 'entregue'),
(2, 'preparando'),
(3, 'pago'),
(4, 'cancelado');

INSERT INTO itens_pedido (
    id_pedido,
    id_produto,
    quantidade,
    preco_unit
) VALUES
(1, 1, 10, 1.20),
(1, 8, 2, 5.00),
(2, 4, 1, 45.00),
(2, 10, 6, 3.50),
(3, 6, 3, 8.50),
(3, 9, 2, 8.00),
(4, 2, 4, 4.50),
(4, 8, 1, 5.00),
(5, 5, 1, 39.90);

COMMIT;
```

## Conferência
```sql
SELECT
    (SELECT COUNT(*) FROM clientes) AS clientes,
    (SELECT COUNT(*) FROM categorias) AS categorias,
    (SELECT COUNT(*) FROM produtos) AS produtos,
    (SELECT COUNT(*) FROM pedidos) AS pedidos,
    (SELECT COUNT(*) FROM itens_pedido) AS itens,
    (SELECT COUNT(*) FROM fornecedores) AS fornecedores,
    (SELECT COUNT(*) FROM produto_fornecedor) AS produto_fornecedor;
```
