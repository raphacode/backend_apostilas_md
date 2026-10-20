# Tabela `clientes`

```sql
CREATE TABLE clientes (
    id_cliente BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    sobrenome VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE,
    telefone VARCHAR(20),
    criado_em TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## Por quê?
- `BIGSERIAL`: gera ID automaticamente.
- `PRIMARY KEY`: identifica cada cliente.
- `VARCHAR(255)`: texto variável.
- `NOT NULL`: nome e sobrenome obrigatórios.
- `UNIQUE`: e-mail não repete.
- e-mail sem `NOT NULL`: pode ser `NULL`.
- telefone é texto porque não fazemos cálculos com ele.
- `TIMESTAMPTZ DEFAULT now()`: data/hora automática.
