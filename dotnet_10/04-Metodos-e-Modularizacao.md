# Capítulo 4 — Métodos e Modularização

> "Não se repita." Métodos são a primeira ferramenta que você aprende para escrever **código limpo e reutilizável**.

---

## 4.1 O que é um método?

Um **método** é um **bloco de código com nome** que executa uma tarefa específica. Você pode "chamá-lo" sempre que precisar dessa tarefa.

### Por que usar métodos?

- **Evita repetição**: escreva uma vez, use cem vezes.
- **Organiza**: cada método faz **uma coisa**, com nome claro.
- **Facilita correção**: se há um bug, você corrige em **um lugar** só.
- **Facilita teste**: dá pra testar cada método isoladamente.

### Comparação cotidiana

Pense em uma receita de bolo:

- **Sem método**: você reescreve o passo "bater os ovos" toda vez que faz qualquer bolo.
- **Com método**: você escreve "Como bater ovos" uma vez e diz "execute *bater ovos*" em cada receita.

---

## 4.2 Anatomia de um método

```csharp
public static int Somar(int a, int b)
{
    int resultado = a + b;
    return resultado;
}
```

| Pedaço | Significado |
|---|---|
| `public` | **Modificador de acesso** (vamos detalhar no Cap. 7). `public` = qualquer um pode chamar. |
| `static` | Vamos explicar no Cap. 7. Por enquanto, é necessário em programas simples. |
| `int` | O **tipo de retorno** — o que o método devolve. |
| `Somar` | O **nome** do método. **PascalCase em C#**: cada palavra começa em maiúscula. |
| `(int a, int b)` | Os **parâmetros** — os "ingredientes" que o método recebe. |
| `{ ... }` | O **corpo** — o que o método faz. |
| `return resultado;` | **Devolve** um valor para quem chamou o método. |

### Chamando o método

```csharp
int total = Somar(3, 5);
Console.WriteLine(total); // 8
```

`3` e `5` são os **argumentos** passados para os parâmetros `a` e `b`.

> **Termos importantes:**
> - **Parâmetro**: a "vaga" que o método declara (`int a`, `int b`).
> - **Argumento**: o **valor real** passado na hora da chamada (`3`, `5`).

---

## 4.3 Métodos sem retorno (`void`)

Quando o método **não devolve nada**, o tipo de retorno é `void`:

```csharp
public static void DarBomDia(string nome)
{
    Console.WriteLine($"Bom dia, {nome}!");
}

// Chamada:
DarBomDia("Maria");
```

`void` significa "vazio". Você não pode escrever `return alguma coisa;` em um método `void` (mas pode escrever apenas `return;` para sair do método antes do fim).

---

## 4.4 Múltiplos parâmetros e nomeados

```csharp
public static double CalcularPreco(double preco, double desconto, double imposto)
{
    return preco - desconto + imposto;
}

// Chamada normal:
double total = CalcularPreco(100, 10, 5);

// Chamada com argumentos nomeados (mais legível!):
double total = CalcularPreco(preco: 100, desconto: 10, imposto: 5);
```

Argumentos nomeados deixam o código **muito mais claro**, especialmente quando há vários parâmetros.

---

## 4.5 Parâmetros opcionais (com valores padrão)

```csharp
public static void EnviarEmail(string para, string assunto = "Sem assunto")
{
    Console.WriteLine($"Para: {para} | Assunto: {assunto}");
}

EnviarEmail("ana@email.com");
// Para: ana@email.com | Assunto: Sem assunto

EnviarEmail("ana@email.com", "Oi!");
// Para: ana@email.com | Assunto: Oi!
```

Parâmetros com valor padrão **devem vir por último** na lista.

---

## 4.6 Sobrecarga de métodos (overloading)

C# permite **vários métodos com o mesmo nome**, desde que tenham **parâmetros diferentes** (em quantidade ou tipo):

```csharp
public static int Somar(int a, int b) => a + b;
public static double Somar(double a, double b) => a + b;
public static int Somar(int a, int b, int c) => a + b + c;
```

O compilador escolhe a versão correta com base nos argumentos passados.

> O `=>` é a "**expression-bodied member**", uma forma curta para métodos de uma só linha.

---

## 4.7 Escopo de variáveis

**Escopo** é onde uma variável **existe** e pode ser usada.

```csharp
public static void Exemplo()
{
    int x = 10;       // visível dentro de Exemplo

    if (x > 5)
    {
        int y = 20;   // visível só dentro do if
        Console.WriteLine(x + y);
    }

    // Console.WriteLine(y); // ERRO! y não existe aqui
}

// Console.WriteLine(x); // ERRO! x não existe fora de Exemplo
```

**Regra geral:** uma variável existe **só dentro do bloco `{ }` onde foi declarada**.

---

## 4.8 Passagem por valor vs. por referência

Por padrão, em C#, os tipos primitivos são passados **por valor** — uma **cópia** é entregue ao método.

```csharp
public static void DobrarValor(int numero)
{
    numero *= 2;
    Console.WriteLine($"Dentro: {numero}");
}

int n = 5;
DobrarValor(n);
Console.WriteLine($"Fora: {n}"); // continua 5!
```

Para passar **por referência** (raro de ser necessário), use `ref`:

```csharp
public static void DobrarValor(ref int numero)
{
    numero *= 2;
}

int n = 5;
DobrarValor(ref n);
Console.WriteLine(n); // 10
```

> Você raramente vai precisar de `ref`. Mencionei aqui só para você reconhecer quando vir.

---

## 4.9 Boas práticas de métodos

1. **Um método deve fazer uma única coisa** (e fazê-la bem).
2. **Nome verbal e descritivo**: `CalcularDesconto`, `EnviarEmail`, `BuscarUsuario`.
3. **Métodos curtos**: idealmente, cabem na tela sem rolar (até ~20 linhas).
4. **Poucos parâmetros**: 3 ou 4 no máximo. Se precisa de muitos, agrupe em uma classe.
5. **Não modifique parâmetros** sem necessidade. Deixe-os "imutáveis" sempre que puder.
6. **Retorne em vez de imprimir** quando possível. Métodos que `WriteLine` são difíceis de testar.

### Exemplo: refatorando

❌ **Ruim**:

```csharp
public static void Calcular()
{
    Console.Write("a: ");
    int a = int.Parse(Console.ReadLine());
    Console.Write("b: ");
    int b = int.Parse(Console.ReadLine());
    int soma = a + b;
    int subtracao = a - b;
    Console.WriteLine($"Soma: {soma}");
    Console.WriteLine($"Subtracao: {subtracao}");
}
```

✅ **Bom**:

```csharp
public static int Somar(int a, int b) => a + b;
public static int Subtrair(int a, int b) => a - b;

public static int LerInteiro(string mensagem)
{
    Console.Write(mensagem);
    return int.Parse(Console.ReadLine());
}

// Uso:
int a = LerInteiro("a: ");
int b = LerInteiro("b: ");
Console.WriteLine($"Soma: {Somar(a, b)}");
Console.WriteLine($"Subtracao: {Subtrair(a, b)}");
```

Cada método faz **uma coisa**, tem nome **claro** e é **reutilizável**.

---

## 4.10 Programa de exemplo completo

```csharp
Console.WriteLine("=== Conversor de Temperatura ===");

double celsius = LerDouble("Digite a temperatura em Celsius: ");

double fahrenheit = CelsiusParaFahrenheit(celsius);
double kelvin = CelsiusParaKelvin(celsius);

Console.WriteLine($"{celsius}°C = {fahrenheit:F2}°F = {kelvin:F2}K");

// --- Métodos ---

static double LerDouble(string mensagem)
{
    Console.Write(mensagem);
    return double.Parse(Console.ReadLine());
}

static double CelsiusParaFahrenheit(double celsius)
{
    return celsius * 9 / 5 + 32;
}

static double CelsiusParaKelvin(double celsius)
{
    return celsius + 273.15;
}
```

> **Observação**: em programas com top-level statements, os métodos auxiliares ficam **no fim do arquivo** e são marcados como `static`.

---

## 4.11 Resumo do capítulo

- **Métodos** organizam o código em pedaços reutilizáveis.
- Têm **tipo de retorno**, **nome**, **parâmetros** e **corpo**.
- `void` indica que o método não retorna nada.
- C# permite **sobrecarga**, **parâmetros opcionais** e **argumentos nomeados**.
- Métodos bons são **curtos**, **com nome claro** e fazem **uma coisa só**.

---

## 4.12 Exercícios

1. Crie um método `EhPar(int numero)` que retorna `true` se for par, `false` se ímpar.
2. Crie um método `CalcularMedia(double[] notas)` que retorna a média de um array de notas.
3. Crie um método `EhMaiorDeIdade(int idade)` que retorna `bool`.
4. Crie um método `Saudacao(string nome, string saudacao = "Olá")` com parâmetro opcional.
5. Refatore o exercício 1 do capítulo anterior (calculadora) para usar métodos para cada operação.

---

➡️ **Próximo capítulo:** [Capítulo 5 — Coleções: Arrays, List e IEnumerable](05-Colecoes-Array-List-IEnumerable.md)
