# Consultas do banco `loja`

## Clientes
```sql
SELECT * FROM clientes ORDER BY id_cliente;
```

## Produtos entre 100 e 500
```sql
SELECT *
FROM produtos
WHERE preco BETWEEN 100 AND 500;
```

## Clientes sem pedidos
```sql
SELECT
    c.id_cliente,
    c.nome,
    c.sobrenome
FROM clientes c
LEFT JOIN pedidos p
    ON p.id_cliente = c.id_cliente
WHERE p.id_pedido IS NULL;
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

## Total do pedido
```sql
SELECT
    i.id_pedido,
    SUM(i.quantidade * i.preco_unit) AS total
FROM itens_pedido i
GROUP BY i.id_pedido
ORDER BY i.id_pedido;
```

## Produtos mais vendidos
```sql
SELECT
    p.id_produto,
    p.nome,
    SUM(i.quantidade) AS quantidade_vendida
FROM produtos p
INNER JOIN itens_pedido i
    ON i.id_produto = p.id_produto
GROUP BY
    p.id_produto,
    p.nome
ORDER BY quantidade_vendida DESC;
```
