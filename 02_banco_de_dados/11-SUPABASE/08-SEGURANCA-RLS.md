---
title: Segurança no Supabase (RLS, senha e chaves)
aliases: [RLS, Row Level Security, Segurança Supabase]
tags: [supabase, seguranca, rls, senha, chaves]
modulo: 11
ordem: 8
---

# 🔒 Segurança: RLS, senha e chaves

> [!WARNING] Por que isso importa
> O Supabase expõe as tabelas do schema `public` por uma **API na internet**. Sem proteção, quem tiver a chave pública pode **ler e escrever** nas suas tabelas.

## 1. Ligar o RLS (Row Level Security)

**RLS** é uma trava por linha: com ela ligada e **sem policies**, a API pública não enxerga nada.

No Query Tool do Supabase:

```sql
ALTER TABLE clientes     ENABLE ROW LEVEL SECURITY;
ALTER TABLE produtos     ENABLE ROW LEVEL SECURITY;
ALTER TABLE pedidos      ENABLE ROW LEVEL SECURITY;
ALTER TABLE itens_pedido ENABLE ROW LEVEL SECURITY;
-- se existirem: categorias, fornecedores, produto_fornecedor
```

> [!NOTE] E o pgAdmin?
> A conexão do pgAdmin usa o usuário `postgres`, que continua funcionando. O RLS bloqueia a **API pública**, não o seu acesso administrativo.

Para ver se está ligado:

```sql
SELECT tablename, rowsecurity
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY tablename;
```

`rowsecurity = true` indica RLS ligado.

## 2. Senha fora do Git

- Guarde a senha em um arquivo `.env`.
- Coloque `.env` no arquivo `.gitignore`.
- **Nunca** suba a senha para o repositório.

```text
# .env  (não vai para o GitHub)
SUPABASE_DB_HOST=...
SUPABASE_DB_USER=postgres.<ID>
SUPABASE_DB_PASSWORD=...
```

```text
# .gitignore
.env
```

## 3. As duas chaves

| Chave | Uso | Regra |
|---|---|---|
| **Publishable** (`sb_publishable_...`) | Navegador / front-end | Segura **só com RLS ligado** |
| **Secret** (`sb_secret_...`) | Servidor | **Nunca** no front-end nem no Git |

> [!DANGER] A chave secret ignora o RLS
> Quem tem a **secret** tem acesso total ao banco. Trate como senha.

## 4. Checklist de segurança

- [ ] RLS ligado em todas as tabelas de `public`.
- [ ] Senha do banco em `.env` e `.env` no `.gitignore`.
- [ ] Chave secret fora de front-end e de repositório.
- [ ] Nenhum print com senha ou chave foi compartilhado.
- [ ] Dados reais de alunos **não** foram colocados no banco de estudo.

Relacionado: [[01-CHECKLIST-SEGURANCA]] e [[03-CHECKLIST-SUPABASE]].

Próximo: [[09-DESAFIOS-SUPABASE]]
