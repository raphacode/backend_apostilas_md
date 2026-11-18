---
title: Exportar o banco local (Backup Plain com INSERT)
aliases: [Exportar padaria.sql, Backup Plain INSERT]
tags: [supabase, backup, plain, insert, pgadmin]
modulo: 11
ordem: 5
---

# 📤 Exportar o banco local

Vamos gerar **um arquivo `padaria.sql`** com as tabelas **e os dados**, usando só a janela **Backup...** do pgAdmin.

> [!INFO] Relação com outro módulo
> É a mesma janela de [[04-BACKUP-SEGURO-E-RESTAURACAO]], mas aqui usamos o formato **Plain** com **INSERT**, porque o Query Tool do Supabase só executa SQL de texto.

## Por que Plain + INSERT?

| Formato | Resultado | Roda no Query Tool? |
|---|---|---|
| **Custom** (`.backup`) | Arquivo binário | **Não** (precisa de Restore) |
| **Plain** com `COPY` (padrão) | Texto, mas com `COPY ... FROM stdin` | **Não** (só no `psql`) |
| **Plain** com **INSERT** | Texto com `CREATE TABLE` e `INSERT` | **Sim** |

> [!WARNING] A caixa que não pode esquecer
> Se **Use INSERT Commands** ficar desligado, os dados vêm em `COPY` e **não executam** no Query Tool.

## Passo a passo

1. Na árvore, abra o servidor **local** e localize o banco `padaria-b-max`.
2. Clique com o **botão direito** no banco → **Backup...**
3. Aba **General**:

| Campo | Valor |
|---|---|
| **Filename** | `padaria.sql` (escolha a pasta pelo ícone de pasta) |
| **Format** | **Plain** |
| **Encoding** | **UTF8** |

4. Aba **Data Options**, seção **Do not save**: ligue **Owner** e **Privileges**.
5. Na mesma aba, deixe desligados **Only data** e **Only schemas**, para o arquivo levar estrutura e dados.
6. Aba **Query Options**:

| Opção | Estado |
|---|---|
| **Use INSERT Commands** | **Ligado** |
| **Include CREATE DATABASE statement** | Desligado |
| **Include DROP DATABASE statement** | Desligado |

7. Clique em **Backup** e espere a mensagem de conclusão.
8. Confirme que o arquivo `padaria.sql` existe na pasta escolhida.

> [!NOTE] Por que desligar Owner e Privileges?
> O dono do banco local (por exemplo, `postgres` da sua máquina) não existe do mesmo jeito no Supabase. Sem Owner e Privileges o arquivo não tenta atribuir donos que dão erro na nuvem.

## Como o banco da aula é

O banco de aula tem **4 tabelas**:

| Tabela | Papel |
|---|---|
| `clientes` | Quem compra |
| `produtos` | O que se vende |
| `pedidos` | Compras (FK para clientes) |
| `itens_pedido` | Itens de cada pedido (FK para pedidos e produtos) |

Se o seu banco tiver as 7 tabelas do [[00-PROJETO-PADARIA]] (com `categorias`, `fornecedores` e `produto_fornecedor`), o mesmo passo a passo serve, e a contagem em [[07-CONFERIR-MIGRACAO]] deve incluir essas tabelas.

## Checklist

- [ ] O formato é **Plain**.
- [ ] O Encoding é **UTF8**.
- [ ] **Use INSERT Commands** está ligado.
- [ ] Owner e Privileges estão em **Do not save**.
- [ ] O arquivo `padaria.sql` foi criado.

Próximo: [[06-IMPORTAR-NO-SUPABASE]]
