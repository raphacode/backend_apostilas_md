# Criar o database e as tabelas da Padaria

## Database
```sql
CREATE DATABASE padaria;
```

Conecte o Query Tool ao banco `padaria` e confirme:

```sql
SELECT current_database();
```

## Tabelas

```sql
CREATE TABLE clientes (
    id_cliente BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    sobrenome VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE,
    telefone VARCHAR(20),
    criado_em TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE categorias (
    id_categoria BIGSERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE produtos (
    id_produto BIGSERIAL PRIMARY KEY,

    id_categoria BIGINT NOT NULL
        REFERENCES categorias(id_categoria),

    nome VARCHAR(255) NOT NULL,

    preco NUMERIC(10,2) NOT NULL
        CHECK (preco >= 0),

    estoque INTEGER NOT NULL DEFAULT 0
        CHECK (estoque >= 0),

    ativo BOOLEAN NOT NULL DEFAULT true,

    criado_em TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE pedidos (
    id_pedido BIGSERIAL PRIMARY KEY,

    id_cliente BIGINT NOT NULL
        REFERENCES clientes(id_cliente),

    status VARCHAR(20) NOT NULL DEFAULT 'aberto'
        CHECK (
            status IN (
                'aberto',
                'pago',
                'preparando',
                'entregue',
                'cancelado'
            )
        ),

    criado_em TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE itens_pedido (
    id_pedido BIGINT NOT NULL
        REFERENCES pedidos(id_pedido)
        ON DELETE CASCADE,

    id_produto BIGINT NOT NULL
        REFERENCES produtos(id_produto),

    quantidade INTEGER NOT NULL
        CHECK (quantidade > 0),

    preco_unit NUMERIC(10,2) NOT NULL
        CHECK (preco_unit >= 0),

    PRIMARY KEY (
        id_pedido,
        id_produto
    )
);

CREATE TABLE fornecedores (
    id_fornecedor BIGSERIAL PRIMARY KEY,
    razao_social VARCHAR(255) NOT NULL,
    cnpj VARCHAR(20) UNIQUE,
    telefone VARCHAR(20),
    ativo BOOLEAN NOT NULL DEFAULT true
);

CREATE TABLE produto_fornecedor (
    id_produto BIGINT NOT NULL
        REFERENCES produtos(id_produto)
        ON DELETE CASCADE,

    id_fornecedor BIGINT NOT NULL
        REFERENCES fornecedores(id_fornecedor)
        ON DELETE CASCADE,

    preco_compra NUMERIC(10,2) NOT NULL
        CHECK (preco_compra >= 0),

    prazo_dias INTEGER NOT NULL DEFAULT 1
        CHECK (prazo_dias >= 0),

    PRIMARY KEY (
        id_produto,
        id_fornecedor
    )
);
```
