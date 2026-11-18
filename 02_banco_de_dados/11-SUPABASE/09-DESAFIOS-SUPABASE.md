---
title: Desafios do módulo Supabase
aliases: [Desafios Supabase, Boss Supabase]
tags: [supabase, desafios, exercicios, boss]
modulo: 11
ordem: 9
---

# 🎮 Desafios do módulo Supabase

Faça na ordem. Cada desafio tem uma **prova** (um resultado que você mostra ao professor).

## Desafios

- [ ] **D1. Conta e projeto**: crie sua conta e o projeto `padaria`. *Prova:* painel aberto com o nome do projeto.
- [ ] **D2. Conexão**: registre o servidor `Supabase` no pgAdmin. *Prova:* `Supabase → Databases → postgres` na árvore.
- [ ] **D3. Migração**: exporte o `padaria.sql` e importe no Supabase. *Prova:* `Query returned successfully`.
- [ ] **D4. Conferência**: rode a contagem das tabelas nos dois bancos. *Prova:* números iguais.
- [ ] **D5. Consulta**: rode uma consulta com `JOIN` e `GROUP BY` no Supabase. *Prova:* resultado igual ao local.

## 👑 Boss

Ligue o **RLS** em todas as tabelas e confira com a consulta de `pg_tables` que `rowsecurity` está `true`. Explique, com suas palavras, **por que** a chave `secret` nunca vai para o front-end.

## 🧪 Teste da sequência

Depois de migrar, o próximo `id` precisa continuar de onde parou, e não voltar para 1. Use um teste dentro de transação (veja [[01-TRANSACOES]]):

```sql
BEGIN;

INSERT INTO clientes (nome)
VALUES ('Teste')
RETURNING id_cliente;

ROLLBACK;
```

> [!NOTE] Adapte as colunas
> A tabela `clientes` pode ter outras colunas obrigatórias (`NOT NULL`). Inclua o que for necessário no `INSERT`. O `ROLLBACK` desfaz o teste e não deixa linha extra.

## Bônus

- [ ] Abra o **Table Editor** no painel do Supabase e veja os dados como planilha.
- [ ] Abra o **SQL Editor** do painel e rode a mesma contagem por lá.

## Save Point

> [!SUCCESS] Módulo concluído
> Se os 5 desafios e o Boss estão marcados, você levou um banco relacional do seu computador para a nuvem, conferiu os dados e o protegeu.

Volte ao início: [[00-SUPABASE-INICIO]]
