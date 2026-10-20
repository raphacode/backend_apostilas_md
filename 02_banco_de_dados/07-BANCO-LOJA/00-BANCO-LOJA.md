# 🏪 Projeto Banco `loja`

```mermaid
erDiagram
    CLIENTES ||--o{ PEDIDOS : realiza
    PEDIDOS ||--|{ ITENS_PEDIDO : possui
    PRODUTOS ||--o{ ITENS_PEDIDO : aparece_em
```

## Tabelas
- [[01-TABELA-CLIENTES]]
- [[02-TABELA-PRODUTOS]]
- [[03-TABELA-PEDIDOS]]
- [[04-TABELA-ITENS-PEDIDO]]
- [[05-POPULAR-LOJA]]
- [[06-CONSULTAS-LOJA]]
- [[08-POPULAR-30-REGISTROS]]
