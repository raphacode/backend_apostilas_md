---
title: Criar conta e projeto no Supabase
aliases: [Criar conta Supabase, Criar projeto Supabase]
tags: [supabase, conta, projeto, passo-a-passo]
modulo: 11
ordem: 2
---

# 🚀 Criar conta e projeto

Cada aluno cria **a sua própria conta**. Assim cada um tem o seu banco, sem mexer no do colega.

## Passo 01 — Criar a conta

1. Abra o navegador e entre em **supabase.com**.
2. Clique em **Start your project** (ou **Sign in** se já tiver conta).
3. Escolha como entrar:
   - **Continue with GitHub** (mais rápido para quem já tem GitHub), ou
   - **e-mail e senha**.
4. Se pedir, confirme o e-mail pela mensagem que chegar na sua caixa de entrada.

> [!NOTE] Organização
> O Supabase cria uma **organização** para a sua conta. Pode deixar o nome sugerido e o tipo **Personal**.

## Passo 02 — Criar o projeto

1. No painel, clique em **New project**.
2. Preencha:

| Campo | O que colocar |
|---|---|
| **Name** | `padaria` (nome livre) |
| **Database Password** | Clique em **Generate a password** ou crie uma forte |
| **Region** | Uma região próxima (a aula usou a dos EUA, `us-east`) |

3. Clique em **Create new project**.
4. Aguarde alguns minutos até o projeto ficar pronto.

> [!DANGER] Guarde a senha agora
> A **Database Password** aparece **uma vez**. Copie para um gerenciador de senhas ou anote em papel. Sem ela, o pgAdmin não conecta. Se perder, dá para redefinir em **Project Settings → Database**, mas isso é trabalho extra.

> [!TIP] Opções da tela de criação
> Na tela de criação existem caixas de opção sobre a **Data API** e sobre o **RLS automático**. Deixe as opções **padrão** e depois confira em [[08-SEGURANCA-RLS]].

## O que você deve ver no fim

- [ ] O painel do projeto aberto, com o nome `padaria`.
- [ ] No menu lateral esquerdo: **Table Editor**, **SQL Editor**, **Database**, **Authentication**, **Storage**.
- [ ] A senha guardada em lugar seguro.

## Mapa do painel

| Menu | Para quê |
|---|---|
| **Table Editor** | Ver e editar tabelas como planilha |
| **SQL Editor** | Escrever SQL no navegador |
| **Database** | Tabelas, extensões, conexões |
| **Authentication** | Usuários e login |
| **Storage** | Arquivos |
| **Project Settings → API Keys** | Chaves publishable e secret |

Próximo: [[03-DADOS-DE-CONEXAO]]
