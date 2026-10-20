# Relacionamentos

## `1:1`
Um registro se relaciona com no máximo um do outro lado.

```text
usuario 1 ─── 1 perfil
```

## `1:N`
```text
cliente 1 ─── N pedidos
```

## `N:N`
```text
pedido N ─── N produto
```

Não implementamos diretamente. Criamos tabela associativa:

```text
pedido 1 ─── N itens_pedido N ─── 1 produto
```

Veja [[04-TABELA-ASSOCIATIVA-N-N]].
