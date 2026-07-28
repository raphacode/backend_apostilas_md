# Capítulo 2 — C# e .NET: Primeiros Passos

> Hora de instalar as ferramentas e escrever seu primeiro programa em C#.

---

## 2.1 As ferramentas que vamos usar

Para programar em C#, você precisa de **três coisas**:

1. **O .NET SDK** — o "kit de desenvolvedor" que contém o compilador e tudo que precisa para criar projetos C#.
2. **Um editor de código** — um programa para escrever o código.
3. **Um terminal** — uma "tela preta" para dar comandos ao computador.

### Editores recomendados

- **Visual Studio Code** (gratuito, leve, multiplataforma) — **vamos usar este na apostila**.
- **Visual Studio Community** (mais robusto, gratuito para uso pessoal, só Windows e Mac).
- **JetBrains Rider** (pago, muito bom).

---

## 2.2 Instalando o .NET SDK

### Passo a passo

1. Acesse: <https://dotnet.microsoft.com/download>
2. Baixe a versão **.NET 8 (LTS)** ou superior.
3. Instale, clicando em "Avançar" até o fim.

### Verificando a instalação

Abra um **terminal** (no Windows: pressione `Win + R`, digite `cmd`, Enter) e digite:

```bash
dotnet --version
```

Se aparecer algo como `8.0.100`, deu certo. Se der "comando não reconhecido", reinicie o computador e tente de novo.

---

## 2.3 Instalando o Visual Studio Code

1. Acesse: <https://code.visualstudio.com/>
2. Baixe e instale.
3. Abra o VS Code.
4. Vá em **Extensions** (ícone de quadradinhos na lateral, ou `Ctrl + Shift + X`).
5. Procure e instale:
   - **C# Dev Kit** (oficial da Microsoft)
   - **C#** (extensão base, geralmente já vem junto)

Pronto. Você está equipado para programar.

---

## 2.4 Seu primeiro programa: "Olá, mundo!"

Existe uma tradição na programação: o primeiro programa que se escreve em qualquer linguagem nova é o **"Olá, mundo!"**. Vamos respeitar a tradição.

### Passo 1: Criar uma pasta

Crie uma pasta no seu computador chamada `meus-estudos-csharp`. Dentro dela, crie outra chamada `01-ola-mundo`.

### Passo 2: Abrir o terminal nessa pasta

No VS Code, abra a pasta `01-ola-mundo` (`File > Open Folder`). Depois, abra o terminal integrado: `Ctrl + '` (crase) ou no menu **Terminal > New Terminal**.

### Passo 3: Criar um projeto C#

No terminal, digite:

```bash
dotnet new console
```

Isso cria um **projeto de console** (programa de linha de comando). Você verá novos arquivos aparecerem na pasta:

- `Program.cs` — o arquivo de código principal.
- `01-ola-mundo.csproj` — o "manifesto" do projeto.

### Passo 4: Olhar o código gerado

Abra o `Program.cs`. Você verá algo assim:

```csharp
// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello, World!");
```

Sim. Só isso. O .NET moderno deixou tudo bem enxuto.

### Passo 5: Rodar o programa

No terminal:

```bash
dotnet run
```

Você verá:

```
Hello, World!
```

🎉 Parabéns. Você acabou de **executar seu primeiro programa em C#**.

---

## 2.5 Entendendo o que aconteceu

Vamos analisar a linha:

```csharp
Console.WriteLine("Hello, World!");
```

| Pedaço | O que é |
|---|---|
| `Console` | Uma "ferramenta" pronta do .NET para interagir com o terminal. |
| `.` (ponto) | Significa "entre dentro de". É como dizer "do `Console`, use o…". |
| `WriteLine` | Um **método** (uma "ação") que escreve uma linha de texto. |
| `("Hello, World!")` | O que se passa entre parênteses é o **argumento**: o texto a ser escrito. |
| `;` (ponto e vírgula) | **Toda instrução em C# termina com ponto e vírgula.** É como o ponto final na frase. |

### Modifique o programa

Troque o texto:

```csharp
Console.WriteLine("Olá, mundo! Estou aprendendo C#!");
```

Salve (`Ctrl + S`) e rode novamente com `dotnet run`. O texto novo deve aparecer.

---

## 2.6 A estrutura "completa" de um programa C#

O que você viu acima é a forma **moderna** e **enxuta** (usando "top-level statements"). Mas é importante conhecer a forma **tradicional**, que você verá em muitos projetos:

```csharp
using System;

namespace MeuPrimeiroPrograma
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Olá, mundo!");
        }
    }
}
```

Vamos quebrar peça por peça (não precisa decorar agora — vai fazer sentido nos próximos capítulos):

| Linha | Significado |
|---|---|
| `using System;` | "Estou importando o pacote `System` para poder usar coisas como `Console`." |
| `namespace MeuPrimeiroPrograma` | Um "sobrenome" para organizar o código (evita conflitos de nomes). |
| `class Program` | Uma **classe** chamada `Program` (vamos aprofundar no Cap. 6). |
| `static void Main(string[] args)` | O **método principal** — o ponto onde o programa começa a executar. |
| `{` e `}` | Marcam o **início e o fim** de blocos de código. |

> Por enquanto, basta saber: **todo programa C# tradicional começa pelo método `Main`**. A versão moderna esconde isso, mas ele continua existindo "por baixo dos panos".

---

## 2.7 Comentários no código

**Comentários** são trechos que o compilador **ignora**. Servem para você explicar para si mesmo (ou para outros programadores) o que o código faz.

```csharp
// Isto é um comentário de uma linha

/* 
   Isto é um comentário
   de várias linhas.
*/

Console.WriteLine("Oi"); // Comentário no fim da linha
```

> **Use comentários com sabedoria.** Um bom código se explica sozinho pelo nome das variáveis e métodos. Comentário só serve para explicar o **porquê** de uma decisão, não o **o quê**.

---

## 2.8 Lendo dados do usuário

Vamos fazer um programa que **conversa** com o usuário:

```csharp
Console.WriteLine("Qual é o seu nome?");
string nome = Console.ReadLine();
Console.WriteLine($"Olá, {nome}! Bem-vindo ao C#!");
```

Rode com `dotnet run` e veja a mágica.

### Explicando

- `Console.ReadLine()` — **lê** uma linha digitada pelo usuário.
- `string nome` — cria uma **variável** do tipo texto chamada `nome`.
- `$"Olá, {nome}!"` — o `$` no início faz com que `{nome}` seja **substituído pelo valor da variável**. Isso se chama **interpolação de strings**.

> Vamos aprofundar variáveis no próximo capítulo. Por ora, observe que **funcionou**.

---

## 2.9 Atalhos úteis no VS Code

| Atalho | Função |
|---|---|
| `Ctrl + S` | Salvar arquivo |
| `Ctrl + '` | Abrir/fechar terminal |
| `Ctrl + .` | **Ações rápidas / sugestões inteligentes** (decora esse, é vida!) |
| `Ctrl + /` | Comentar/descomentar linha |
| `F2` | Renomear variável/método em todo o projeto |
| `Ctrl + Shift + P` | Abrir paleta de comandos |
| `Alt + Shift + F` | Formatar o código |

> O `Ctrl + .` é especialmente importante. Ele oferece **sugestões automáticas**: criar um método que ainda não existe, importar um `using`, gerar um construtor… Use-o sempre.

---

## 2.10 Estrutura de pastas comum em um projeto

Quando você roda `dotnet new console`, aparecem estes itens:

```
01-ola-mundo/
├── bin/                    ← onde fica o programa compilado (ignore)
├── obj/                    ← arquivos temporários da compilação (ignore)
├── Program.cs              ← seu código principal
└── 01-ola-mundo.csproj     ← configuração do projeto
```

**As pastas `bin` e `obj` são geradas automaticamente.** Você nunca edita nada lá. Toda vez que você roda `dotnet build` ou `dotnet run`, elas são (re)criadas.

### O arquivo `.csproj`

É um arquivo **XML** que descreve seu projeto: versão do .NET, dependências, configurações. Por enquanto não precisa mexer nele.

---

## 2.11 Comandos `dotnet` básicos que você vai usar muito

| Comando | O que faz |
|---|---|
| `dotnet new console` | Cria um projeto de console |
| `dotnet new webapi` | Cria uma API web (vamos usar na Parte IV) |
| `dotnet build` | Compila o projeto (verifica erros) |
| `dotnet run` | Compila **e** executa |
| `dotnet add package <nome>` | Adiciona uma biblioteca externa |
| `dotnet --info` | Mostra informações sobre a instalação do .NET |

---

## 2.12 Resumo do capítulo

- Instalamos o **.NET SDK** e o **VS Code**.
- Criamos um projeto com `dotnet new console`.
- Rodamos com `dotnet run`.
- Aprendemos sobre `Console.WriteLine`, `Console.ReadLine`, comentários e interpolação de strings.
- Conhecemos o atalho mágico **`Ctrl + .`**.

---

## 2.13 Exercícios

1. Crie um novo projeto chamado `02-saudacao` e escreva um programa que:
   - Pergunta o nome do usuário.
   - Pergunta a idade do usuário.
   - Imprime uma mensagem: `"Olá, [nome]! Você tem [idade] anos."`
2. Modifique o programa para perguntar também a cidade.
3. Use o `Ctrl + .` em alguma palavra do seu código e veja o que aparece. Não precisa aplicar nenhuma sugestão — só explore.

---

➡️ **Próximo capítulo:** [Capítulo 3 — Lógica de Programação com C#](03-Logica-de-Programacao-com-CSharp.md)
