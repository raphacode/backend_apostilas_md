# Checklist de Segurança

Antes de `UPDATE`:
- [ ] Estou no database correto?
- [ ] Conferi `SELECT current_database();`?
- [ ] Testei o `WHERE` com `SELECT`?
- [ ] Sei quantas linhas serão alteradas?
- [ ] Selecionei o `WHERE` junto?
- [ ] Posso usar `BEGIN`?

Antes de `DELETE`:
- [ ] Existe `WHERE`?
- [ ] Existem FKs?
- [ ] Existe `ON DELETE CASCADE`?
- [ ] Entendo o efeito em cascata?

Antes de `TRUNCATE`/`DROP`:
- [ ] É banco de teste?
- [ ] Tenho backup?
- [ ] Tenho certeza?
