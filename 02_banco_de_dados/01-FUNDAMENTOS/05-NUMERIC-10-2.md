# `NUMERIC(10,2)`

```sql
preco NUMERIC(10,2) NOT NULL CHECK (preco >= 0)
```

- `10`: quantidade total de dígitos;
- `2`: casas decimais;
- `NOT NULL`: obrigatório;
- `CHECK (preco >= 0)`: impede preço negativo.

Maior valor positivo aproximado:

```text
99999999.99
```

Para dinheiro, `NUMERIC` é apropriado porque representa decimais com precisão exata.
