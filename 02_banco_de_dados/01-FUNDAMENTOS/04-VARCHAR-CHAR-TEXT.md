# `VARCHAR`, `CHAR` e `TEXT`

## `VARCHAR(n)`
Texto variável com limite máximo.

```sql
nome VARCHAR(255)
```

Ele não preenche o restante até 255.

## `CHAR(n)`
Comprimento fixo. Valores menores são preenchidos com espaços.

```text
CHAR(10) contendo ABC
→ ABC_______
```

No PostgreSQL, `CHAR` não deve ser tratado como escolha automática de otimização.

## `TEXT`
Texto variável sem limite declarado pelo desenvolvedor.

## Regra prática
- nome → `VARCHAR`;
- email → `VARCHAR`;
- descrição longa → `TEXT`;
- código realmente fixo → pode usar `CHAR`.
