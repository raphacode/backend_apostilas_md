---
title: Conferir a migração
aliases: [Conferir Supabase, Contagem das tabelas]
tags: [supabase, conferencia, count, validacao]
modulo: 11
ordem: 7
---

# ✅ Conferir a migração

Migrar não é terminar: é **provar** que os dados chegaram iguais.

## 1. Ver as tabelas

1. Na árvore: **Supabase → Databases → postgres → Schemas → public → Tables**.
2. Clique com o botão direito em **Tables → Refresh**.
3. Devem aparecer: `clientes`, `itens_pedido`, `pedidos`, `produtos`.

## 2. Contar as linhas

Abra um **Query Tool no Supabase** (confira `@Supabase` no título) e rode:

```sql
SELECT 'clientes' AS tabela, COUNT(*) FROM clientes
UNION ALL SELECT 'produtos', COUNT(*) FROM produtos
UNION ALL SELECT 'pedidos', COUNT(*) FROM pedidos
UNION ALL SELECT 'itens_pedido', COUNT(*) FROM itens_pedido;
```

> [!SUCCESS] Resultado esperado
> Rode a **mesma consulta** no banco local `padaria-b-max`. Os números precisam ser **idênticos**.

| Tabela | Local | Supabase | Igual? |
|---|---|---|---|
| `clientes` | | | [ ] |
| `produtos` | | | [ ] |
| `pedidos` | | | [ ] |
| `itens_pedido` | | | [ ] |

## 3. Testar uma consulta das aulas

Use uma consulta de [[01-JOINS]] ou de [[03-GROUP-BY-AGREGACOES]] e compare o resultado com o local.

## 4. Conferir o banco conectado

```sql
SELECT current_database() AS banco_conectado;
```

No Supabase o resultado é `postgres`.

> [!TIP] Se faltar linha
> Confira se você executou o arquivo **inteiro** (F5 sem selecionar parte do texto). Se só um trecho estiver selecionado, o pgAdmin executa apenas ele.

Próximo: [[08-SEGURANCA-RLS]]
