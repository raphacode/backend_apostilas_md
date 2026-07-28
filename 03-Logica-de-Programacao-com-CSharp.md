# Capítulo 3 — Lógica de Programação com C#

> Neste capítulo você aprende a **pensar como programador**. Variáveis, decisões e repetições são a base de tudo.

---

## 3.1 O que é uma variável?

Uma **variável** é um **espaço na memória** do computador onde guardamos uma informação para usar depois. Pense numa **caixinha com etiqueta**:

- A **etiqueta** é o nome da variável (`nome`, `idade`, `preco`).
- O **conteúdo** da caixinha é o valor (`"Maria"`, `25`, `19.90`).

### Declarando variáveis em C#

```csharp
string nome = "Maria";
int idade = 25;
double preco = 19.90;
bool estaLogado = true;
```

Em C#, você precisa **dizer o tipo** da variável (`string`, `int`, `double`, `bool`) **antes** do nome. Isso é o que chamamos de linguagem **fortemente tipada**.

> **Por que tipar é bom?** Porque o compilador pega muitos erros antes de o programa rodar. Se você tentar somar texto com número, ele avisa.

### Regras para nomear variáveis

- Comece com letra ou `_` (não pode começar com número).
- Não pode ter espaço nem acento.
- **Use camelCase**: `nomeDoUsuario`, `precoTotal`, `quantidadeDeProdutos`.
- O nome deve **descrever o que guarda**. Evite `x`, `a`, `tmp`. Prefira `idade`, `quantidadeDeAlunos`.

```csharp
// Ruim
int x = 25;

// Bom
int idadeDoUsuario = 25;
```

### Atribuição vs. comparação

- `=` significa **"recebe"** (atribuição).
- `==` significa **"é igual a?"** (comparação).

```csharp
int idade = 18;        // recebe
bool eMaiorDeIdade = idade == 18;   // é igual a?
```

---

## 3.2 Tipos de dados primitivos

Os principais tipos em C#:

| Tipo | O que guarda | Exemplo |
|---|---|---|
| `int` | Número inteiro | `42`, `-7`, `0` |
| `long` | Número inteiro **grande** | `9999999999L` |
| `double` | Número decimal (alta precisão) | `3.14`, `19.90` |
| `decimal` | Número decimal **muito preciso** (use para dinheiro) | `19.90m` |
| `float` | Número decimal (menos preciso) | `3.14f` |
| `bool` | Verdadeiro ou falso | `true`, `false` |
| `char` | Um único caractere | `'a'`, `'?'` |
| `string` | Texto (vários caracteres) | `"olá"` |
| `DateTime` | Data e hora | `DateTime.Now` |

### Pegadinhas importantes

- `string` usa **aspas duplas**: `"texto"`. `char` usa **aspas simples**: `'a'`.
- `double` aceita decimais com ponto, não vírgula: `19.90` (não `19,90`).
- Para **dinheiro**, **sempre** use `decimal` (mais preciso e exigido em sistemas financeiros).
- `decimal` precisa do sufixo `m`: `decimal preco = 19.90m;`.
- `float` precisa do sufixo `f`: `float pi = 3.14f;`.

### `var` — deixar o compilador descobrir

Você pode escrever:

```csharp
var nome = "Maria";   // o compilador entende que é string
var idade = 25;       // entende que é int
```

O `var` é **prático**, mas só funciona quando você atribui um valor na hora. **Não é variável "sem tipo"** — o tipo continua existindo, só é deduzido automaticamente.

---

## 3.3 Constantes

Quando um valor **nunca vai mudar**, use `const`:

```csharp
const double PI = 3.14159;
const string NOME_DA_EMPRESA = "Acme Corp";
```

Tente alterar uma constante e o compilador vai dar erro.

---

## 3.4 Operadores

### Operadores aritméticos

| Operador | Significado | Exemplo | Resultado |
|---|---|---|---|
| `+` | Soma | `5 + 3` | `8` |
| `-` | Subtração | `5 - 3` | `2` |
| `*` | Multiplicação | `5 * 3` | `15` |
| `/` | Divisão | `10 / 2` | `5` |
| `%` | Resto da divisão (módulo) | `10 % 3` | `1` |

> **Cuidado com a divisão de inteiros**: `7 / 2` em C# dá `3` (inteiro), não `3.5`. Para obter `3.5`, faça `7.0 / 2` ou `7 / 2.0`.

### Operadores de atribuição

```csharp
int x = 10;
x += 5;   // o mesmo que x = x + 5  → x agora é 15
x -= 3;   // 12
x *= 2;   // 24
x /= 4;   // 6
```

### Operadores de comparação

| Operador | Significado |
|---|---|
| `==` | Igual a |
| `!=` | Diferente de |
| `>` | Maior que |
| `<` | Menor que |
| `>=` | Maior ou igual |
| `<=` | Menor ou igual |

Sempre retornam `true` ou `false`.

### Operadores lógicos

| Operador | Significado |
|---|---|
| `&&` | E (todas as condições verdadeiras) |
| `\|\|` | OU (pelo menos uma verdadeira) |
| `!` | NÃO (inverte) |

```csharp
bool maiorDeIdade = idade >= 18;
bool temCNH = true;
bool podeDirigir = maiorDeIdade && temCNH;   // true só se ambos
```

### Concatenação de strings

```csharp
string nome = "Maria";
string saudacao = "Olá, " + nome + "!";

// Melhor (interpolação):
string saudacao2 = $"Olá, {nome}!";
```

**Prefira a interpolação** (`$""`). Fica mais limpo.

---

## 3.5 Estruturas condicionais (`if`/`else`)

Condicionais são o jeito de o programa **tomar decisões**.

### `if` simples

```csharp
int idade = 20;

if (idade >= 18)
{
    Console.WriteLine("Maior de idade.");
}
```

A condição vai **entre parênteses**. O bloco que executa vai **entre chaves**.

### `if` / `else`

```csharp
if (idade >= 18)
{
    Console.WriteLine("Maior de idade.");
}
else
{
    Console.WriteLine("Menor de idade.");
}
```

### `if` / `else if` / `else`

```csharp
int nota = 75;

if (nota >= 90)
{
    Console.WriteLine("A");
}
else if (nota >= 80)
{
    Console.WriteLine("B");
}
else if (nota >= 70)
{
    Console.WriteLine("C");
}
else
{
    Console.WriteLine("Reprovado");
}
```

### Boas práticas

- **Sempre** use chaves `{ }`, mesmo para um único comando. Evita bugs.
- Não aninhe muitos `if`s. Se passar de 2 ou 3 níveis, há algo errado.

### Operador ternário

Forma curta de um `if/else` simples:

```csharp
string mensagem = idade >= 18 ? "Maior" : "Menor";
```

Lê-se: *"se idade ≥ 18, então `Maior`, senão `Menor`."*

---

## 3.6 `switch` — quando há muitas opções

Quando você compara **uma variável com vários valores**, prefira `switch`:

```csharp
string dia = "segunda";

switch (dia)
{
    case "segunda":
        Console.WriteLine("Início da semana");
        break;
    case "sexta":
        Console.WriteLine("Quase fim de semana!");
        break;
    case "sabado":
    case "domingo":
        Console.WriteLine("Fim de semana!");
        break;
    default:
        Console.WriteLine("Dia normal");
        break;
}
```

- `break` é **obrigatório** ao final de cada caso.
- `default` é o "senão" (executa se nenhum caso bate).

### `switch` expression (forma moderna)

```csharp
string mensagem = dia switch
{
    "segunda" => "Início da semana",
    "sexta" => "Quase fim de semana!",
    "sabado" or "domingo" => "Fim de semana!",
    _ => "Dia normal"
};
```

Mais limpo. Você verá muito em código moderno.

---

## 3.7 Laços de repetição (loops)

Quando precisamos **repetir** uma ação várias vezes, usamos **loops**.

### `while` — enquanto a condição for verdadeira

```csharp
int contador = 1;

while (contador <= 5)
{
    Console.WriteLine($"Contagem: {contador}");
    contador++;   // muito importante! senão é loop infinito
}
```

### `do/while` — executa pelo menos uma vez

```csharp
int numero;

do
{
    Console.WriteLine("Digite um número entre 1 e 10:");
    numero = int.Parse(Console.ReadLine());
}
while (numero < 1 || numero > 10);
```

A condição é checada **depois** da primeira execução.

### `for` — quando você sabe quantas vezes vai repetir

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"i vale {i}");
}
```

Quebrando o `for`:

- `int i = 0` — **inicialização** (executa uma vez antes de tudo).
- `i < 5` — **condição** (testada antes de cada repetição).
- `i++` — **passo** (executa após cada repetição).

> `i++` é o mesmo que `i = i + 1`. Outros: `i--` (decrementa), `i += 2` (incrementa de 2 em 2).

### `foreach` — percorrer uma coleção

```csharp
string[] nomes = { "Maria", "João", "Ana" };

foreach (string nome in nomes)
{
    Console.WriteLine($"Olá, {nome}");
}
```

**Sempre que possível, prefira `foreach` a `for`** ao percorrer coleções. É mais legível.

### `break` e `continue`

- `break` — **sai** do loop imediatamente.
- `continue` — **pula** para a próxima iteração.

```csharp
for (int i = 1; i <= 10; i++)
{
    if (i == 5) break;       // para no 5
    if (i % 2 == 0) continue; // pula pares
    Console.WriteLine(i);
}
// Saída: 1, 3
```

### Cuidado com loops infinitos

```csharp
while (true)
{
    Console.WriteLine("Socorro!"); // nunca para!
}
```

Sempre garanta que a condição **vai eventualmente ficar falsa**.

---

## 3.8 Conversão entre tipos

Às vezes você precisa **converter** um tipo em outro.

### Conversão implícita (segura)

```csharp
int x = 10;
double y = x;   // tudo bem, int cabe em double
```

### Conversão explícita (cast)

```csharp
double pi = 3.14;
int aproximado = (int)pi;   // vira 3 (perde os decimais!)
```

### Convertendo string em número

```csharp
string textoIdade = "25";
int idade = int.Parse(textoIdade);
double preco = double.Parse("19,90");

// Mais seguro: TryParse
if (int.TryParse(textoIdade, out int resultado))
{
    Console.WriteLine($"Convertido: {resultado}");
}
else
{
    Console.WriteLine("Não é um número válido");
}
```

> Use `TryParse` quando o valor vem do usuário. Ele **não quebra** se o usuário digitar besteira.

### Convertendo número em string

```csharp
int idade = 25;
string texto = idade.ToString();
```

---

## 3.9 Trabalhando com texto (`string`)

`string` tem dezenas de métodos úteis. Os mais usados:

```csharp
string nome = "Maria Silva";

nome.Length;              // 11 (tamanho)
nome.ToUpper();           // "MARIA SILVA"
nome.ToLower();           // "maria silva"
nome.Trim();              // tira espaços nas pontas
nome.Contains("Silva");   // true
nome.StartsWith("Maria"); // true
nome.Replace("Silva", "Souza"); // "Maria Souza"
nome.Split(' ');          // ["Maria", "Silva"]
nome.Substring(0, 5);     // "Maria"
```

> **Importante**: `string` é **imutável** em C#. Os métodos não mudam a string original — eles **retornam uma nova**. Você precisa **guardar** o resultado: `nome = nome.ToUpper();`.

---

## 3.10 Programa completo de exemplo

```csharp
Console.WriteLine("=== Calculadora de IMC ===");

Console.Write("Digite seu peso (kg): ");
double peso = double.Parse(Console.ReadLine());

Console.Write("Digite sua altura (m): ");
double altura = double.Parse(Console.ReadLine());

double imc = peso / (altura * altura);

Console.WriteLine($"Seu IMC é {imc:F2}");

if (imc < 18.5)
{
    Console.WriteLine("Abaixo do peso");
}
else if (imc < 25)
{
    Console.WriteLine("Peso normal");
}
else if (imc < 30)
{
    Console.WriteLine("Sobrepeso");
}
else
{
    Console.WriteLine("Obesidade");
}
```

`{imc:F2}` formata o número com **2 casas decimais**.

---

## 3.11 Resumo do capítulo

- **Variáveis** guardam valores; cada uma tem um **tipo**.
- **Operadores** fazem contas, comparações e combinações lógicas.
- **`if/else`** toma decisões. **`switch`** lida com muitos casos.
- **Loops** (`while`, `for`, `foreach`) repetem ações.
- Sempre converta strings para números com **`TryParse`** quando vier do usuário.

---

## 3.12 Exercícios

1. Faça um programa que peça dois números e mostre **soma, subtração, multiplicação e divisão**.
2. Faça um programa que peça a idade e diga se a pessoa é **criança (0-12), adolescente (13-17), adulto (18-59) ou idoso (60+)**.
3. Faça um programa que imprima a **tabuada de um número** digitado pelo usuário (de 1 a 10).
4. Faça um programa que peça **5 números** e calcule a **média**.
5. Faça um programa que peça um número e diga se ele é **par ou ímpar**.
6. Faça um programa que peça uma frase e mostre quantas **palavras** ela tem (dica: use `Split`).

---

➡️ **Próximo capítulo:** [Capítulo 4 — Métodos e Modularização](04-Metodos-e-Modularizacao.md)
