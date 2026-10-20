# Modelagem da Padaria

## Cliente → Pedido
```text
1:N
```

## Categoria → Produto
```text
1:N
```

## Pedido ↔ Produto
```text
N:N
```

Resolvido por `itens_pedido`.

## Produto ↔ Fornecedor
```text
N:N
```

Resolvido por `produto_fornecedor`.

Uma tabela associativa pode guardar atributos do relacionamento, por exemplo:
- preço de compra;
- prazo de entrega.
