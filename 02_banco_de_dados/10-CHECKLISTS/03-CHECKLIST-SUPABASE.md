---
title: Checklist Supabase
aliases: [Checklist migração Supabase]
tags: [supabase, checklist, migracao]
---

# Checklist Supabase

Módulo completo: [[00-SUPABASE-INICIO]]

Antes de migrar:
- [ ] Tenho conta e projeto no Supabase? ([[02-CRIAR-CONTA-E-PROJETO]])
- [ ] Guardei a senha do banco em lugar seguro?
- [ ] Anotei host do **Session pooler**, porta `5432` e usuário `postgres.<ID>`? ([[03-DADOS-DE-CONEXAO]])
- [ ] O servidor `Supabase` aparece no pgAdmin? ([[04-CONECTAR-PGADMIN]])

Ao exportar:
- [ ] Formato **Plain** e Encoding **UTF8**?
- [ ] **Use INSERT Commands** ligado?
- [ ] Owner e Privileges em **Do not save**? ([[05-EXPORTAR-BANCO-LOCAL]])

Ao importar:
- [ ] O título da aba do Query Tool mostra `@Supabase`?
- [ ] Apaguei as linhas `\restrict` e `\unrestrict`?
- [ ] Executei o arquivo inteiro com F5? ([[06-IMPORTAR-NO-SUPABASE]])

Depois de migrar:
- [ ] A contagem das tabelas é igual no local e no Supabase? ([[07-CONFERIR-MIGRACAO]])
- [ ] RLS ligado em todas as tabelas? ([[08-SEGURANCA-RLS]])
- [ ] Senha em `.env` e `.env` no `.gitignore`?
- [ ] Chave `secret` fora do front-end e do Git?
