# `UPDATE` e riscos reais

## Correto
```sql
UPDATE clientes
SET telefone = '21999566120'
WHERE id_cliente = 1;
```

## Perigoso
```sql
UPDATE clientes
SET telefone = '21999566120';
```

Sem `WHERE`, todas as linhas são alteradas.

## Regra de ouro
Antes do `UPDATE`, teste o mesmo filtro:

```sql
SELECT *
FROM clientes
WHERE id_cliente = 1;
```

Depois:

```sql
UPDATE clientes
SET telefone = '21999566120'
WHERE id_cliente = 1
RETURNING *;
```

## Danos possíveis em empresas
- preços errados;
- contatos sobrescritos;
- estoque incorreto;
- dados históricos alterados;
- pedidos associados ao cliente errado;
- indisponibilidade;
- prejuízo financeiro.

## Não altere PK sem necessidade
```sql
UPDATE clientes
SET id_cliente = 500
WHERE id_cliente = 1;
```

Evite isso.
