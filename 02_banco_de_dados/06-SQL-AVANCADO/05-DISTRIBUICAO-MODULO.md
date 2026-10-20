# Distribuindo produtos com módulo

```sql
INNER JOIN produtos AS produto
    ON produto.id_produto =
       ((pedido.id_pedido - 1 + item.deslocamento) % 12) + 1
```

Objetivo: gerar IDs entre 1 e 12.

## Pedido 1
Deslocamentos 0, 4 e 8:
```text
produto 1
produto 5
produto 9
```

## Pedido 2
```text
produto 2
produto 6
produto 10
```

O `% 12` faz o ciclo voltar ao início depois do 12.

> [!WARNING]
> Essa técnica é excelente para dados de teste controlados, mas presume IDs contínuos de 1 a 12.
