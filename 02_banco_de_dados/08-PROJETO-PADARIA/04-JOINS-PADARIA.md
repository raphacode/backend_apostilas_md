# JOINs da Padaria

## Pedido + cliente
```sql
SELECT
    p.id_pedido,
    c.nome,
    c.sobrenome,
    p.status
FROM pedidos p
INNER JOIN clientes c
    ON c.id_cliente = p.id_cliente;
```

## Todos os clientes, mesmo sem pedido
```sql
SELECT
    c.id_cliente,
    c.nome,
    p.id_pedido
FROM clientes c
LEFT JOIN pedidos p
    ON p.id_cliente = c.id_cliente;
```

## Produto + categoria
```sql
SELECT
    p.id_produto,
    p.nome AS produto,
    c.nome AS categoria,
    p.preco
FROM produtos p
INNER JOIN categorias c
    ON c.id_categoria = p.id_categoria;
```

## Pedido completo
```sql
SELECT
    pe.id_pedido,
    cli.nome AS cliente,
    pro.nome AS produto,
    i.quantidade,
    i.preco_unit,
    i.quantidade * i.preco_unit AS subtotal
FROM pedidos pe
INNER JOIN clientes cli
    ON cli.id_cliente = pe.id_cliente
INNER JOIN itens_pedido i
    ON i.id_pedido = pe.id_pedido
INNER JOIN produtos pro
    ON pro.id_produto = i.id_produto
ORDER BY pe.id_pedido, pro.nome;
```

## N:N produto ↔ fornecedor
```sql
SELECT
    p.nome AS produto,
    f.razao_social AS fornecedor,
    pf.preco_compra,
    pf.prazo_dias
FROM produto_fornecedor pf
INNER JOIN produtos p
    ON p.id_produto = pf.id_produto
INNER JOIN fornecedores f
    ON f.id_fornecedor = pf.id_fornecedor
ORDER BY p.nome, pf.preco_compra;
```
