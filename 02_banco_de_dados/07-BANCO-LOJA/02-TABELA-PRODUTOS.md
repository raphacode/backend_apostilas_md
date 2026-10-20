# Tabela `produtos`

```sql
CREATE TABLE produtos (
    id_produto BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    preco NUMERIC(10,2) NOT NULL CHECK (preco >= 0),
    estoque INTEGER NOT NULL DEFAULT 0 CHECK (estoque >= 0),
    ativo BOOLEAN NOT NULL DEFAULT true,
    criado_em TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Regras:
- preço obrigatório e não negativo;
- estoque obrigatório e não negativo;
- ativo começa `true`;
- data automática.
