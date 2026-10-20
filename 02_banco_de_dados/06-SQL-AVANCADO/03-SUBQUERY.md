# Subquery

Subquery é uma consulta dentro de outra.

```sql
SELECT
    (SELECT COUNT(*) FROM clientes) AS total_clientes,
    (SELECT COUNT(*) FROM produtos) AS total_produtos,
    (SELECT COUNT(*) FROM pedidos) AS total_pedidos;
```

Cada subconsulta retorna um único valor.

`AS total_clientes` cria um nome amigável para a coluna resultante.
