# `generate_series`, `%`, `LPAD`, `::TEXT`

## Série
```sql
SELECT *
FROM generate_series(1,10);
```

## `%` é módulo
É o resto da divisão.

```sql
12 % 10
```

Resultado:
```text
2
```

## Ciclo 1 a 10
```sql
((numero - 1) % 10) + 1
```

Produz:
```text
1,2,3,4,5,6,7,8,9,10,1,2...
```

## Conversão
```sql
numero::TEXT
```

equivale a:
```sql
CAST(numero AS TEXT)
```

## `LPAD`
```sql
LPAD(numero::TEXT, 2, '0')
```

```text
1 → 01
9 → 09
10 → 10
```

## Concatenação
```sql
'Silva' || ' ' || LPAD(1::TEXT, 2, '0')
```

Resultado:
```text
Silva 01
```
