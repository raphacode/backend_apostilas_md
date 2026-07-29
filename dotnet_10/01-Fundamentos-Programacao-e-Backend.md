# Capítulo 1 — Fundamentos de Programação e Backend

> "Antes de aprender a escrever código, você precisa entender **o que** está escrevendo e **para quem**."

Este capítulo não tem uma única linha de código. E está tudo bem. Você vai entender o **mundo** em que o programador vive, e isso vai fazer todo o resto da apostila fazer sentido.

---

## 1.1 O que é programação?

Programação é o ato de **dar instruções para um computador realizar uma tarefa**.

O computador, sozinho, não sabe fazer nada. Ele só executa, em altíssima velocidade, ordens muito específicas que alguém escreveu para ele.

### Comparação fácil

Imagine que você quer ensinar uma criança muito pequena (que não conhece nada do mundo) a fazer um sanduíche. Você teria que dizer:

1. Pegue duas fatias de pão.
2. Abra a geladeira.
3. Pegue o queijo.
4. Coloque uma fatia de queijo sobre uma fatia de pão.
5. Coloque a outra fatia de pão por cima.

Se você pular um passo, a criança não faz. Se você falar "faz um sanduíche", ela não entende.

**Programar é exatamente isso:** descrever, **passo a passo**, o que o computador deve fazer. A diferença é que, no lugar do português, usamos uma **linguagem de programação** (no nosso caso, **C#**).

### Por que precisamos de uma "linguagem de programação"?

O computador, internamente, só entende **0 e 1** (chamado de linguagem de máquina ou binário). Escrever 0 e 1 o dia inteiro seria insano. Por isso, criaram-se linguagens **mais próximas do humano** (como C#, Python, JavaScript). Um programa especial chamado **compilador** traduz o que escrevemos para o que o computador entende.

> **Em uma frase:** programar é traduzir a sua ideia para uma sequência de instruções que o computador consegue executar.

---

## 1.2 O que é Backend?

Quase todo aplicativo ou site moderno tem **duas partes principais**:

| Parte | O que é | Onde roda |
|---|---|---|
| **Frontend** | A parte **visual**, com a qual o usuário interage | No celular, no navegador, no computador do usuário |
| **Backend** | A parte dos **bastidores**, que faz a "mágica" acontecer | Em um servidor, longe do usuário |

### Analogia do restaurante

Pense em um **restaurante**:

- **Frontend = o salão.** Tem mesas bonitas, cardápio, garçons, decoração. É o que o cliente vê e toca.
- **Backend = a cozinha.** O cliente nem entra lá. É onde a comida é realmente preparada, onde estão os ingredientes, os fogões, os cozinheiros.

Quando você pede um prato, o garçom (frontend) leva o pedido para a cozinha (backend). A cozinha prepara e devolve o prato pronto. O cliente nunca conversa diretamente com a cozinha — ele só vê o resultado final no salão.

### O que o Backend faz?

- **Guarda dados** (em um banco de dados): usuários, produtos, mensagens, fotos.
- **Processa regras de negócio**: calcular um desconto, validar um CPF, descontar saldo de uma conta.
- **Garante segurança**: quem pode ver o quê, quem pode editar.
- **Conecta com outros sistemas**: meios de pagamento, envio de e-mail, etc.
- **Responde ao frontend** quando ele pede alguma coisa.

> **Em uma frase:** Backend é o "cérebro" do sistema. O frontend é o "rosto".

---

## 1.3 Diferença entre Frontend e Backend

| Aspecto | Frontend | Backend |
|---|---|---|
| **Onde executa** | Navegador / celular do usuário | Servidor remoto |
| **Quem vê** | Todo mundo (visual) | Ninguém (invisível) |
| **Foco** | Aparência, experiência, interação | Dados, regras, segurança, performance |
| **Linguagens comuns** | HTML, CSS, JavaScript | C#, Java, Python, Node.js, Go |
| **Exemplo** | A tela do Instagram | O servidor do Instagram que guarda as fotos |

### Exemplo prático: postar uma foto no Instagram

1. **Frontend** (app no celular): você tira uma foto, escreve uma legenda e aperta "Postar".
2. O frontend **envia** a foto e a legenda **pelo Internet** para o backend do Instagram.
3. **Backend** (servidor): recebe a foto, salva em um banco de dados, faz uma cópia menor para a thumbnail, registra que aquela foto é sua, atualiza o feed dos seus amigos.
4. **Backend** responde dizendo "ok, deu certo".
5. **Frontend** mostra a foto no seu perfil.

Tudo isso acontece em segundos. **Sem backend, o Instagram não existiria** — a foto sumiria assim que você fechasse o app.

---

## 1.4 O que é C#?

**C#** (lê-se "C Sharp", como uma nota musical) é uma **linguagem de programação** criada pela **Microsoft** em **2000**, junto com o **.NET**.

### Características importantes

- **Moderna**: tem todos os recursos esperados de uma linguagem atual.
- **Fortemente tipada**: você precisa dizer claramente que tipo de informação cada variável guarda. Isso evita muitos erros.
- **Orientada a objetos**: organiza o código em "objetos" (veremos no Capítulo 6).
- **Multiplataforma**: roda no Windows, Linux e Mac.
- **Versátil**: dá pra fazer **backend de APIs**, jogos (com Unity), aplicativos desktop, mobile, IoT, IA…
- **Madura e bem documentada**: tem milhares de tutoriais, livros e cursos.

### Por que aprender C# para backend?

- É **uma das linguagens mais usadas no mercado** corporativo, especialmente em empresas grandes.
- Tem **ótimo desempenho** (rápido).
- Tem ferramentas excelentes (Visual Studio, VS Code, Rider).
- A **comunidade é enorme**.
- O **mercado de trabalho paga bem** para quem domina C#/.NET.

---

## 1.5 O que é .NET?

Aqui vem uma confusão comum entre iniciantes. Vamos separar:

- **C#** = a **linguagem** (a sintaxe, as palavras-chave, as regras).
- **.NET** = a **plataforma** (o conjunto de ferramentas, bibliotecas e o ambiente que executa o seu código).

### Comparação

Pense em **C#** como o **idioma** (português, inglês). Pense em **.NET** como o **país** com toda a infraestrutura (estradas, hospitais, lojas) que permite você viver usando esse idioma.

### O que vem dentro do .NET?

- **Runtime (CLR)**: o programa que **executa** seu código C#. É como o "motor".
- **Bibliotecas (BCL)**: milhares de funções **prontas** para você usar (ler arquivos, fazer requisições HTTP, manipular datas, etc.).
- **SDK**: o "kit de desenvolvedor" — ferramentas para criar, compilar, rodar e publicar projetos.
- **ASP.NET Core**: a parte do .NET focada em **construir aplicações web e APIs**. Vamos usá-la na Parte IV.

### Versões do .NET

Você vai ouvir falar de ".NET Framework" (o antigo, só Windows) e ".NET moderno" (multiplataforma, versão atual nesta apostila: .NET 10). **Nesta apostila vamos usar a versão moderna.**

---

## 1.6 Para que serve C# no Backend?

C# no backend serve para:

- **Construir APIs** (mais comum hoje em dia).
- Conectar com **bancos de dados** (SQL Server, PostgreSQL, MySQL, MongoDB…).
- Implementar **regras de negócio** complexas.
- Processar **arquivos**, gerar **relatórios**, enviar **e-mails**.
- Integrar com **outros sistemas** (gateways de pagamento, serviços de nuvem).
- **Autenticar usuários** e proteger informações.

Empresas como **Microsoft, Stack Overflow, Accenture, bancos brasileiros** e muitas outras usam C# no backend de seus sistemas.

---

## 1.7 O que é uma aplicação backend?

Uma **aplicação backend** é um programa que:

1. Fica **rodando o tempo todo** em um computador chamado **servidor**.
2. **Espera** receber pedidos vindos pela internet.
3. Quando recebe um pedido, **processa**, geralmente conversa com o **banco de dados**, e **responde**.
4. **Nunca dorme**: precisa estar disponível 24/7.

Exemplo: o backend do Spotify recebe milhões de pedidos por segundo ("toca essa música", "salva essa playlist"), busca as informações, e responde para cada celular pedindo.

---

## 1.8 O que é um servidor?

Um **servidor** é, **fisicamente**, um computador. Mas é um computador **diferente do seu**:

- Não tem mouse, nem teclado, nem tela.
- Fica em **data centers** (galpões cheios de computadores).
- Tem **conexão com a internet** muito rápida.
- Roda **24 horas por dia, 7 dias por semana**.
- Geralmente roda **Linux** (sim, mesmo aplicações C# rodam em Linux hoje).

> **Servidor é só um computador que serve outras pessoas.** Daí o nome.

Quando você acessa `google.com`, o seu computador (cliente) faz um pedido para um servidor do Google. O servidor responde com a página. Essa relação se chama **cliente-servidor**.

### Onde ficam os servidores hoje?

Na maior parte dos casos, em **provedores de nuvem** como:

- **Microsoft Azure**
- **Amazon Web Services (AWS)**
- **Google Cloud**

Você "aluga" um servidor virtual deles em vez de comprar uma máquina física.

---

## 1.9 O que é uma API?

**API** = **Application Programming Interface** — em português, **Interface de Programação de Aplicações**.

Calma, esse nome é assustador. Vamos com calma.

### Definição simples

> **Uma API é uma "porta de entrada" que um sistema oferece para que outros sistemas conversem com ele.**

### Analogia do garçom (essa é clássica)

- O **cliente** (você) quer comer.
- A **cozinha** (backend) sabe cozinhar.
- O **garçom** (API) é o intermediário: ele recebe seu pedido, leva pra cozinha, traz a comida pronta.

Você, cliente, **não precisa entrar na cozinha**. Você só precisa saber o que pedir ao garçom (o cardápio = a documentação da API).

### Outro exemplo

Quando você usa um app de previsão do tempo:

1. O **app no seu celular** (frontend) pergunta para uma **API de meteorologia**: "qual a temperatura em São Paulo agora?"
2. A **API** responde: "23°C, parcialmente nublado".
3. O app mostra isso na tela.

A API é a **forma padronizada** de fazer essa pergunta e receber essa resposta.

> **Em uma frase:** uma API é um conjunto de regras e endereços que define como dois sistemas trocam informações.

---

## 1.10 O que é Requisição e Resposta?

Toda comunicação com uma API funciona em **dois movimentos**:

### Requisição (Request)

É o **pedido** que o cliente faz ao servidor. Contém:

- **Endereço** (URL): para onde o pedido vai.
- **Método** (GET, POST…): que tipo de ação se deseja.
- **Cabeçalhos** (headers): informações extras (idioma, autenticação…).
- **Corpo** (body): dados enviados (opcional).

### Resposta (Response)

É o que o servidor **devolve**. Contém:

- **Status code** (200, 404, 500…): código que indica se deu certo ou errado.
- **Cabeçalhos**.
- **Corpo**: a informação solicitada (geralmente em **JSON**).

### Exemplo concreto

**Requisição:**
```
GET https://api.exemplo.com/usuarios/42
```
*"Servidor, me dá os dados do usuário de número 42."*

**Resposta:**
```json
{
  "id": 42,
  "nome": "Maria",
  "email": "maria@exemplo.com"
}
```
*"Aqui estão os dados, com sucesso (status 200)."*

Veremos tudo isso em profundidade no **Capítulo 9**.

---

## 1.11 Resumo do capítulo

- **Programar** é dar instruções, passo a passo, para o computador.
- **Frontend** é o que o usuário vê; **Backend** é o que faz a "mágica" acontecer nos bastidores.
- **C#** é a linguagem; **.NET** é a plataforma que executa C#.
- **Servidor** é um computador que fica ligado o tempo todo, recebendo pedidos.
- **API** é a porta de entrada do backend.
- Toda comunicação envolve **requisição** (pedido) e **resposta**.

---

## 1.12 Exercícios de fixação (sem código)

Responda mentalmente ou em um caderno:

1. Em suas palavras, o que é programação?
2. Qual a diferença entre frontend e backend?
3. Cite três coisas que o backend faz.
4. Qual a diferença entre C# e .NET?
5. O que é um servidor?
6. Explique o que é uma API usando uma analogia diferente da do garçom.
7. O que é uma requisição? O que é uma resposta?
8. Por que estudar C# pode ser uma boa escolha de carreira?

---

➡️ **Próximo capítulo:** [Capítulo 2 — C# e .NET: Primeiros Passos](02-CSharp-e-DotNet-Primeiros-Passos.md)
