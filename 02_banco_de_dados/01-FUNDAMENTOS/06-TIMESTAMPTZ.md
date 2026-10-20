# `TIMESTAMPTZ NOT NULL DEFAULT now()`

```sql
criado_em TIMESTAMPTZ NOT NULL DEFAULT now()
```

- `TIMESTAMPTZ`: data e hora com tratamento de fuso;
- `DEFAULT now()`: preenche automaticamente;
- `NOT NULL`: impede ausência de valor.

Mesmo com `DEFAULT`, o `NOT NULL` protege contra tentativa explícita de inserir `NULL`.
