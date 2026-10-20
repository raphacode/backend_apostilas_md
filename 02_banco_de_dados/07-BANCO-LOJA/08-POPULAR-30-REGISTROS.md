# Popular banco `loja` — 30 registros por tabela

> [!WARNING]
> Script para banco de aula. Ele apaga os registros atuais das quatro tabelas e reinicia os IDs. Execute-o somente quando puder descartar os dados existentes.

## Resultado esperado

| Tabela | Registros após o script |
|---|---:|
| `clientes` | 30 |
| `produtos` | 30 |
| `pedidos` | 30 |
| `itens_pedido` | 30 |

## Ordem de execução

O script respeita as chaves estrangeiras nesta sequência:

```text
clientes + produtos → pedidos → itens_pedido
```

Para que `itens_pedido` também tenha exatamente 30 registros, cada pedido criado recebe um único item. Em uma aplicação real, um pedido pode ter vários itens.

## Script de população

```sql
BEGIN;

-- ============================================================
-- LIMPEZA DAS TABELAS
-- ============================================================
-- Remove os dados anteriores,
-- reinicia os BIGSERIALs
-- e respeita as dependências entre as tabelas.

TRUNCATE TABLE
    itens_pedido,
    pedidos,
    produtos,
    clientes
RESTART IDENTITY CASCADE;


-- ============================================================
-- 1. CLIENTES
-- Cria 30 clientes
-- ============================================================

WITH dados_clientes AS (
    SELECT
        ARRAY[
            'Ana',
            'Bruno',
            'Carla',
            'Diego',
            'Elisa',
            'Felipe',
            'Gabriela',
            'Henrique',
            'Isabela',
            'Joao'
        ] AS nomes,

        ARRAY[
            'Silva',
            'Lima',
            'Souza',
            'Santos',
            'Oliveira',
            'Costa',
            'Rocha',
            'Martins',
            'Ferreira',
            'Almeida'
        ] AS sobrenomes
)

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
        || '@loja.com',

    '1199'
        || LPAD(serie.numero::TEXT, 7, '0'),

    now() - make_interval(
        days => 31 - serie.numero
    )

FROM generate_series(1, 30) AS serie(numero)

CROSS JOIN dados_clientes;


-- ============================================================
-- 2. PRODUTOS
-- Cadastra 30 produtos de exemplo
-- ============================================================

INSERT INTO produtos (
    nome,
    preco,
    estoque,
    ativo,
    criado_em
)

SELECT
    catalogo.nome,
    catalogo.preco,
    catalogo.estoque,
    catalogo.ativo,

    now() - make_interval(
        days => 31 - catalogo.posicao
    )

FROM (
    VALUES
        ( 1, 'Mouse sem fio',           79.90,   90, true),
        ( 2, 'Teclado mecanico',       189.90,   45, true),
        ( 3, 'Monitor 24 polegadas',   899.00,   25, true),
        ( 4, 'Notebook 15 polegadas', 3499.90,   12, true),
        ( 5, 'Webcam Full HD',         219.90,   38, true),
        ( 6, 'Headset USB',            249.90,   42, true),
        ( 7, 'Pen drive 64 GB',         69.90,  120, true),
        ( 8, 'SSD 480 GB',             329.90,   55, true),
        ( 9, 'Cadeira gamer',         1299.90,   18, true),
        (10, 'Impressora tanque',      749.90,   20, true),
        (11, 'Roteador Wi-Fi',         299.90,   36, true),
        (12, 'Caixa de som',           159.90,   64, true),
        (13, 'Cabo HDMI 2 m',           39.90,  140, true),
        (14, 'Adaptador USB-C',         89.90,   75, true),
        (15, 'Microfone USB',          319.90,   30, true),
        (16, 'Mousepad grande',         59.90,  110, true),
        (17, 'HD externo 1 TB',        449.90,   32, true),
        (18, 'Suporte para notebook',  129.90,   50, true),
        (19, 'Hub USB 4 portas',       119.90,   67, true),
        (20, 'Luminaria LED',           99.90,   80, true),
        (21, 'Fonte notebook',         279.90,   22, true),
        (22, 'Memoria RAM 8 GB',       239.90,   48, true),
        (23, 'Placa de video',        1899.90,   10, true),
        (24, 'Filtro de linha',         69.90,  100, true),
        (25, 'Mochila para notebook',  199.90,   40, true),
        (26, 'Base refrigerada',       149.90,   33, true),
        (27, 'Leitor de cartao',        49.90,   86, true),
        (28, 'Cabo de rede 5 m',        29.90,  150, true),
        (29, 'Organizador de cabos',    24.90,  200, false),
        (30, 'Suporte para monitor',   179.90,   27, true)

) AS catalogo(
    posicao,
    nome,
    preco,
    estoque,
    ativo
);


-- ============================================================
-- 3. PEDIDOS
-- Cria 30 pedidos
-- Um pedido para cada cliente
-- ============================================================

WITH clientes_ordenados AS (

    SELECT
        id_cliente,

        ROW_NUMBER() OVER (
            ORDER BY id_cliente
        ) AS posicao

    FROM clientes
)

INSERT INTO pedidos (
    id_cliente,
    status,
    criado_em
)

SELECT
    cliente.id_cliente,

    CASE ((cliente.posicao - 1) % 4)

        WHEN 0 THEN 'aberto'

        WHEN 1 THEN 'pago'

        WHEN 2 THEN 'enviado'

        ELSE 'cancelado'

    END,

    -- CORREÇÃO:
    -- ROW_NUMBER() retorna BIGINT.
    -- make_interval(days => ...) precisa receber INTEGER.
    -- Por isso fazemos o CAST usando ::INTEGER.

    now() - make_interval(
        days => (31 - cliente.posicao)::INTEGER
    )

FROM clientes_ordenados AS cliente

WHERE cliente.posicao <= 30;


-- ============================================================
-- 4. ITENS_PEDIDO
-- Cria 30 itens
-- ============================================================
-- O pedido da posição N será associado
-- ao produto da posição N.
--
-- preco_unit recebe o preço do produto
-- no momento em que o item é criado.

WITH pedidos_ordenados AS (

    SELECT
        id_pedido,

        ROW_NUMBER() OVER (
            ORDER BY id_pedido
        ) AS posicao

    FROM pedidos
),

produtos_ordenados AS (

    SELECT
        id_produto,
        preco,

        ROW_NUMBER() OVER (
            ORDER BY id_produto
        ) AS posicao

    FROM produtos
)

INSERT INTO itens_pedido (
    id_pedido,
    id_produto,
    quantidade,
    preco_unit
)

SELECT
    pedido.id_pedido,

    produto.id_produto,

    ((pedido.posicao - 1) % 5) + 1,

    produto.preco

FROM pedidos_ordenados AS pedido

INNER JOIN produtos_ordenados AS produto
    ON produto.posicao = pedido.posicao

WHERE pedido.posicao <= 30;


-- ============================================================
-- FINALIZA A TRANSAÇÃO
-- ============================================================

COMMIT;
```

## Conferência das quantidades

Execute após o script. Cada resultado deve ser `30`.

```sql
SELECT 'clientes' AS tabela, COUNT(*) AS total FROM clientes
UNION ALL
SELECT 'produtos', COUNT(*) FROM produtos
UNION ALL
SELECT 'pedidos', COUNT(*) FROM pedidos
UNION ALL
SELECT 'itens_pedido', COUNT(*) FROM itens_pedido
ORDER BY tabela;
```

## Conferência dos relacionamentos

Esta consulta mostra os 30 itens criados com o cliente, pedido, produto, quantidade e total de cada linha.

```sql
SELECT
    pedido.id_pedido,
    cliente.nome || ' ' || cliente.sobrenome AS cliente,
    pedido.status,
    produto.nome AS produto,
    item.quantidade,
    item.preco_unit,
    item.quantidade * item.preco_unit AS total_item
FROM itens_pedido AS item
INNER JOIN pedidos AS pedido
    ON pedido.id_pedido = item.id_pedido
INNER JOIN clientes AS cliente
    ON cliente.id_cliente = pedido.id_cliente
INNER JOIN produtos AS produto
    ON produto.id_produto = item.id_produto
ORDER BY pedido.id_pedido;
```

## Consulta extra: total por pedido

```sql
SELECT
    pedido.id_pedido,
    cliente.nome || ' ' || cliente.sobrenome AS cliente,
    pedido.status,
    SUM(item.quantidade * item.preco_unit) AS total_pedido
FROM pedidos AS pedido
INNER JOIN clientes AS cliente
    ON cliente.id_cliente = pedido.id_cliente
INNER JOIN itens_pedido AS item
    ON item.id_pedido = pedido.id_pedido
GROUP BY
    pedido.id_pedido,
    cliente.nome,
    cliente.sobrenome,
    pedido.status
ORDER BY pedido.id_pedido;
```
