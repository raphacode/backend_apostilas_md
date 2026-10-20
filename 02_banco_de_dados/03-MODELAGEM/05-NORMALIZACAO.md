# Normalização — visão prática

Evite:

```text
pedido
cliente_nome
cliente_email
produto1
produto2
produto3
```

Prefira:

```text
clientes
pedidos
produtos
itens_pedido
```

Assim cada entidade possui sua própria tabela e os vínculos usam PK/FK.

A normalização ajuda a evitar duplicidade, inconsistência e manutenção difícil.
