---
title: Dados de conexão (Session pooler)
aliases: [Connection string Supabase, Session pooler, Dados de conexão]
tags: [supabase, conexao, pooler, host, porta]
modulo: 11
ordem: 3
---

# 🔌 Dados de conexão

Para o pgAdmin falar com o Supabase você precisa de **4 informações**: host, porta, usuário e senha.

## Onde encontrar

1. No painel do seu projeto, clique no botão **Connect** (parte de cima da tela).
2. Escolha o método **Session pooler**.
3. Anote os dados que aparecem.

## O que anotar

| Campo | Exemplo (fictício) | Observação |
|---|---|---|
| **Host** | `aws-0-us-east-2.pooler.supabase.com` | Termina em `.pooler.supabase.com` |
| **Port** | `5432` | Porta do Session pooler |
| **Database** | `postgres` | Sempre `postgres` |
| **User** | `postgres.abcdefghijklmnop` | **`postgres.` + o ID do seu projeto** |
| **Password** | a que você criou | A senha do banco |

> [!WARNING] O usuário não é só `postgres`
> No pooler, o usuário é `postgres.` seguido do **ID do projeto** (project ref). Se usar só `postgres`, a conexão falha.

## Por que Session pooler?

O Supabase oferece 3 caminhos:

| Método | Porta | Quando usar |
|---|---|---|
| **Direct connection** | 5432 | Só funciona se a sua rede tiver **IPv6** |
| **Session pooler** | 5432 | **Use este.** Funciona em IPv4 e atende o pgAdmin |
| **Transaction pooler** | 6543 | Aplicações serverless; não é para o pgAdmin |

> [!SUCCESS] Resultado da aula
> Na turma, o caminho que funcionou foi o **Session pooler**. O host direto (`db.<projeto>.supabase.co`) falhou com `Network is unreachable`, porque a rede não tinha IPv6.

## Anotação para a aula

```text
Host:     ______________________________.pooler.supabase.com
Porta:    5432
Banco:    postgres
Usuário:  postgres.______________________
Senha:    (guardada em lugar seguro, não escreva aqui)
```

> [!CAUTION] Não publique
> Não coloque a senha em arquivo do GitHub nem em print compartilhado. Veja [[08-SEGURANCA-RLS]].

Próximo: [[04-CONECTAR-PGADMIN]]
