# Popular completamente o banco `loja`

> [!WARNING]
> Script para banco de aula. Ele apaga os dados existentes e reinicia os IDs.

```sql
-- BEGIN inicia uma transação para executar todo o carregamento como uma única operação.
BEGIN;

-- TRUNCATE apaga os registros das tabelas; RESTART IDENTITY reinicia os IDs e CASCADE inclui tabelas dependentes.
TRUNCATE TABLE
    itens_pedido,
    pedidos,
    produtos,
    clientes
RESTART IDENTITY CASCADE;

-- WITH cria uma fonte temporária com listas de nomes e sobrenomes para gerar clientes.
WITH dados_clientes AS (
    SELECT
        ARRAY[
            'Ana','Bruno','Carla','Diego','Elisa',
            'Felipe','Gabriela','Henrique','Isabela','Joao'
        ] AS nomes,

        ARRAY[
            'Silva','Lima','Souza','Santos','Oliveira',
            'Costa','Rocha','Martins','Ferreira','Almeida'
        ] AS sobrenomes
)

-- INSERT INTO clientes grava os 50 clientes gerados pelo SELECT.
INSERT INTO clientes (
    nome,
    sobrenome,
    email,
    telefone,
    criado_em
)
SELECT
    dados_clientes.nomes[
        ((serie.numero - 1) % 10) + 1
    ],

    dados_clientes.sobrenomes[
        ((serie.numero - 1) % 10) + 1
    ]
    || ' '
    || LPAD(serie.numero::TEXT, 2, '0'),

    'cliente'
        || LPAD(serie.numero::TEXT, 2, '0')
        || '@email.com',

    '2199'
        || LPAD(serie.numero::TEXT, 7, '0'),

    now() - make_interval(days => 50 - serie.numero)

FROM generate_series(1, 50) AS serie(numero)
CROSS JOIN dados_clientes;

-- INSERT INTO produtos cadastra o catálogo inicial com preço, estoque e situação de cada produto.
INSERT INTO produtos (nome, preco, estoque, ativo) VALUES
('Mouse', 59.90, 100, true),
('Teclado', 129.90, 80, true),
('Monitor', 899.00, 30, true),
('Notebook', 3499.90, 15, true),
('Webcam', 189.90, 45, true),
('Headset', 249.90, 60, true),
('Pen Drive 64 GB', 69.90, 120, true),
('SSD 480 GB', 329.90, 50, true),
('Cadeira Gamer', 1299.90, 20, true),
('Impressora', 749.90, 25, true),
('Roteador', 299.90, 40, true),
('Caixa de Som', 159.90, 70, true);

-- INSERT INTO pedidos cria 70 pedidos, associando clientes, status e datas diferentes.
INSERT INTO pedidos (id_cliente, status, criado_em)
SELECT
    ((numero - 1) % 35) + 1,

    CASE ((numero - 1) % 4)
        WHEN 0 THEN 'aberto'
        WHEN 1 THEN 'pago'
        WHEN 2 THEN 'enviado'
        ELSE 'cancelado'
    END,

    now() - make_interval(hours => numero)

FROM generate_series(1, 70) AS serie(numero);

-- INSERT INTO itens_pedido cria os produtos e quantidades associados a cada pedido.
INSERT INTO itens_pedido (
    id_pedido,
    id_produto,
    quantidade,
    preco_unit
)
SELECT
    pedido.id_pedido,
    produto.id_produto,

    CASE item.posicao
        WHEN 1 THEN ((pedido.id_pedido - 1) % 3) + 1
        WHEN 2 THEN (pedido.id_pedido % 3) + 1
        ELSE ((pedido.id_pedido + 1) % 3) + 1
    END,

    produto.preco

FROM pedidos AS pedido

CROSS JOIN (
    VALUES
        (1, 0),
        (2, 4),
        (3, 8)
) AS item(posicao, deslocamento)

INNER JOIN produtos AS produto
    ON produto.id_produto =
       ((pedido.id_pedido - 1 + item.deslocamento) % 12) + 1;

-- COMMIT confirma a transação e torna todas as inserções permanentes.
COMMIT;
```

## Conferência
```sql
-- A consulta de conferência conta quantos registros foram criados em cada tabela.
SELECT
    (SELECT COUNT(*) FROM clientes) AS total_clientes,
    (SELECT COUNT(*) FROM produtos) AS total_produtos,
    (SELECT COUNT(*) FROM pedidos) AS total_pedidos,
    (SELECT COUNT(*) FROM itens_pedido) AS total_itens_pedido;
```
