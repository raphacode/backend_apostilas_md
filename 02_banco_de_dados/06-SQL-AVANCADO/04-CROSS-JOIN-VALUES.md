# `CROSS JOIN` e `VALUES`

## `VALUES`
```sql
VALUES
    (1,0),
    (2,4),
    (3,8)
```

Com:
```sql
AS item(posicao, deslocamento)
```

vira conceitualmente:

| posicao | deslocamento |
|---:|---:|
| 1 | 0 |
| 2 | 4 |
| 3 | 8 |

## `CROSS JOIN`
Combina cada linha de uma origem com todas as linhas da outra.

70 pedidos × 3 linhas = 210 combinações.

```sql
FROM pedidos AS pedido
CROSS JOIN (
    VALUES
        (1,0),
        (2,4),
        (3,8)
) AS item(posicao, deslocamento)
```
