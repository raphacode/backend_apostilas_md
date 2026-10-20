# O que é banco de dados?

Banco de dados é uma estrutura organizada para armazenar informações.

Exemplos:
- clientes;
- produtos;
- pedidos;
- estoque;
- fornecedores;
- pagamentos.

## SGBD
Sistema Gerenciador de Banco de Dados. Exemplos: PostgreSQL, MySQL, SQL Server, Oracle, SQLite.

## Banco relacional
Os dados ficam em tabelas relacionadas.

```text
clientes 1 ─── N pedidos
pedidos  1 ─── N itens_pedido
produtos 1 ─── N itens_pedido
```

Veja [[02-TABELA-LINHA-COLUNA]].
