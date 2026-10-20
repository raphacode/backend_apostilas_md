# Relatórios da Padaria

## Total de cada pedido
```sql
SELECT
    i.id_pedido,
    SUM(i.quantidade * i.preco_unit) AS total_pedido
FROM itens_pedido i
GROUP BY i.id_pedido
ORDER BY i.id_pedido;
```

## Pedidos por cliente
```sql
SELECT
    c.id_cliente,
    c.nome,
    COUNT(p.id_pedido) AS quantidade_pedidos
FROM clientes c
LEFT JOIN pedidos p
    ON p.id_cliente = c.id_cliente
GROUP BY
    c.id_cliente,
    c.nome
ORDER BY quantidade_pedidos DESC;
```

## Produtos mais vendidos
```sql
SELECT
    p.id_produto,
    p.nome,
    SUM(i.quantidade) AS unidades_vendidas
FROM produtos p
INNER JOIN itens_pedido i
    ON i.id_produto = p.id_produto
GROUP BY
    p.id_produto,
    p.nome
ORDER BY unidades_vendidas DESC;
```

## Estoque por categoria
```sql
SELECT
    c.nome AS categoria,
    SUM(p.estoque) AS estoque_total
FROM categorias c
INNER JOIN produtos p
    ON p.id_categoria = c.id_categoria
GROUP BY
    c.id_categoria,
    c.nome
ORDER BY estoque_total DESC;
```

## Clientes com pelo menos 2 pedidos
```sql
SELECT
    c.id_cliente,
    c.nome,
    COUNT(p.id_pedido) AS quantidade
FROM clientes c
INNER JOIN pedidos p
    ON p.id_cliente = c.id_cliente
GROUP BY
    c.id_cliente,
    c.nome
HAVING COUNT(p.id_pedido) >= 2;
```

## Menor preço de fornecedor por produto
```sql
SELECT
    p.id_produto,
    p.nome,
    MIN(pf.preco_compra) AS menor_preco_compra
FROM produtos p
INNER JOIN produto_fornecedor pf
    ON pf.id_produto = p.id_produto
GROUP BY
    p.id_produto,
    p.nome
ORDER BY menor_preco_compra;
```
