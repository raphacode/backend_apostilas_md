# Capítulo 5 — Coleções: Arrays, List e IEnumerable

> Quase todo programa real precisa lidar com **vários itens ao mesmo tempo**: lista de usuários, produtos, mensagens. As **coleções** são para isso.

---

## 5.1 O problema que coleções resolvem

Imagine que você tem **5 alunos** e quer guardar o nome de cada um. Sem coleções:

```csharp
string aluno1 = "Maria";
string aluno2 = "João";
string aluno3 = "Ana";
string aluno4 = "Pedro";
string aluno5 = "Lucas";
```

E se forem **500 alunos**? E se você não souber **quantos** vão ser? Caos.

A solução: usar uma **coleção** — uma única variável que guarda **vários valores**.

---

## 5.2 Arrays

Um **array** é a coleção mais simples e antiga. Tem **tamanho fixo** definido na criação.

### Declarando um array

```csharp
// Tamanho fixo, valores padrão (0 para int)
int[] idades = new int[5];

// Com valores iniciais
string[] nomes = { "Maria", "João", "Ana" };

// Forma equivalente
string[] nomes2 = new string[] { "Maria", "João", "Ana" };
```

### Acessando elementos

Os arrays são **indexados a partir de 0**:

```csharp
string[] nomes = { "Maria", "João", "Ana" };

Console.WriteLine(nomes[0]); // Maria
Console.WriteLine(nomes[1]); // João
Console.WriteLine(nomes[2]); // Ana

nomes[1] = "Joana";          // alterando

Console.WriteLine(nomes.Length); // 3
```

> **Cuidado**: acessar `nomes[3]` quando só há 3 elementos (índices 0, 1, 2) gera o erro `IndexOutOfRangeException`.

### Percorrendo um array

```csharp
// Com for
for (int i = 0; i < nomes.Length; i++)
{
    Console.WriteLine(nomes[i]);
}

// Com foreach (mais limpo)
foreach (string nome in nomes)
{
    Console.WriteLine(nome);
}
```

### Limitação dos arrays

- **Tamanho fixo**: você não pode adicionar nem remover itens depois.
- Para "adicionar" um item, você teria que criar um array novo, maior, e copiar os antigos.

Por isso, na prática, **quase sempre usamos `List<T>`**.

---

## 5.3 List<T> — a coleção mais usada

`List<T>` é uma lista **dinâmica** (cresce e diminui sozinha). O `<T>` significa "tipo genérico" — você define que tipo a lista vai guardar.

### Criando uma List

```csharp
using System.Collections.Generic; // necessário no .NET tradicional

List<string> nomes = new List<string>();

// Ou com valores iniciais
List<int> numeros = new List<int> { 1, 2, 3, 4, 5 };

// Sintaxe moderna no .NET 10 (C# 14; recurso introduzido no C# 12)
List<int> numeros2 = [1, 2, 3, 4, 5];
```

### Adicionando, removendo e acessando

```csharp
List<string> alunos = new List<string>();

alunos.Add("Maria");
alunos.Add("João");
alunos.Add("Ana");

Console.WriteLine(alunos[0]);     // Maria (igual a array)
Console.WriteLine(alunos.Count);  // 3 (note: Count, não Length)

alunos.Remove("João");            // remove pelo valor
alunos.RemoveAt(0);               // remove pelo índice
alunos.Insert(0, "Carlos");       // insere na posição 0
alunos.Clear();                   // remove todos
```

### Métodos úteis de List

```csharp
List<int> numeros = new List<int> { 5, 2, 8, 1, 9, 3 };

numeros.Contains(8);   // true
numeros.IndexOf(8);    // 2
numeros.Sort();        // [1, 2, 3, 5, 8, 9]
numeros.Reverse();     // inverte
numeros.Count;         // 6
```

### Percorrendo

```csharp
foreach (int numero in numeros)
{
    Console.WriteLine(numero);
}
```

### List é o "padrão"

> **Regra prática:** se você não tem um motivo específico para usar outra coleção, **use `List<T>`**. É flexível, intuitiva e atende 90% dos casos.

---

## 5.4 IEnumerable<T>

Aqui chegamos a um conceito **crucial** que muitos iniciantes não entendem direito. Vamos com calma.

### O que é IEnumerable?

`IEnumerable<T>` é uma **interface** (vamos detalhar interfaces no Cap. 8). Por enquanto, pense nela como um **contrato mínimo**:

> "Eu sou algo que você pode **percorrer com `foreach`** — e só isso."

### Por que isso importa?

`IEnumerable<T>` é o **denominador comum** de todas as coleções em C#. Tanto `T[]` quanto `List<T>`, `HashSet<T>`, `Dictionary<T,U>.Values` etc. **são** `IEnumerable<T>`.

```csharp
int[] array = { 1, 2, 3 };
List<int> lista = new List<int> { 1, 2, 3 };

IEnumerable<int> a = array; // funciona
IEnumerable<int> b = lista; // funciona
```

### O que IEnumerable **não** te dá

Comparado com `List<T>`, `IEnumerable<T>` **não** tem:

- `Add`, `Remove`, `Clear` (não dá para modificar)
- Indexador (`coleção[0]`)
- `Count` (tem `Count()`, mas é um método caro — explico abaixo)

Você só pode **iterar** com `foreach` e usar métodos do **LINQ** (`Where`, `Select`, etc.).

---

## 5.5 Diferença entre `List<T>` e `IEnumerable<T>`

| Característica | `List<T>` | `IEnumerable<T>` |
|---|---|---|
| Pode adicionar/remover? | ✅ Sim | ❌ Não |
| Acesso por índice (`[i]`)? | ✅ Sim | ❌ Não |
| Sabe o tamanho rapidamente? | ✅ `Count` (instantâneo) | ⚠️ `Count()` percorre toda a coleção |
| Carrega tudo na memória? | ✅ Sim | ⚠️ Pode ser **lazy** (carrega sob demanda) |
| Para que serve? | Manipular dados | **Apenas ler / iterar** |

### A grande sacada: lazy evaluation

`IEnumerable<T>` pode ser **preguiçoso** ("lazy"). O conteúdo só é gerado **na hora em que você itera**.

Exemplo prático:

```csharp
IEnumerable<int> numeros = Enumerable.Range(1, 1_000_000_000);
// Isso NÃO cria 1 bilhão de números na memória!

foreach (int n in numeros.Take(5))
{
    Console.WriteLine(n); // 1, 2, 3, 4, 5 — gerados sob demanda
}
```

Se isso fosse uma `List<int>`, você estouraria a memória.

### Quando usar cada um

#### Use `List<T>` quando:

- Vai **adicionar/remover** itens.
- Precisa **acessar por índice** (`lista[5]`).
- Vai **percorrer múltiplas vezes** (com `IEnumerable` lazy, cada iteração reexecuta a lógica).
- Está **construindo uma coleção** dentro do método.

#### Use `IEnumerable<T>` quando:

- O método só precisa **ler** os dados.
- Você quer **flexibilidade** (aceita array, lista, qualquer coisa).
- Trabalha com **streams de dados** ou **consultas LINQ** (banco de dados).
- Quer permitir **lazy evaluation**.

### Boa prática para parâmetros e retornos

```csharp
// ❌ Limita demais quem pode chamar
public static void Imprimir(List<string> nomes) { ... }

// ✅ Aceita qualquer coleção
public static void Imprimir(IEnumerable<string> nomes) { ... }
```

> **Regra**: receba o **mais geral possível** (`IEnumerable<T>`), retorne o **mais específico que faça sentido**.

---

## 5.6 Outras coleções importantes

Citando rapidamente — você vai encontrar pela frente:

| Coleção | Para que serve |
|---|---|
| `Dictionary<K, V>` | Pares chave-valor: `dicionario["maria"] = 25`. |
| `HashSet<T>` | Conjunto sem duplicatas, busca rapidíssima. |
| `Queue<T>` | Fila (FIFO — primeiro a entrar, primeiro a sair). |
| `Stack<T>` | Pilha (LIFO — último a entrar, primeiro a sair). |

### Exemplo de Dictionary

```csharp
Dictionary<string, int> idades = new Dictionary<string, int>();
idades["Maria"] = 25;
idades["João"] = 30;

Console.WriteLine(idades["Maria"]); // 25

if (idades.ContainsKey("Ana"))
{
    Console.WriteLine(idades["Ana"]);
}

foreach (KeyValuePair<string, int> par in idades)
{
    Console.WriteLine($"{par.Key} tem {par.Value} anos");
}
```

---

## 5.7 LINQ — uma prévia

LINQ é uma **linguagem de consulta** integrada ao C#. Funciona em qualquer `IEnumerable<T>`.

```csharp
using System.Linq;

List<int> numeros = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

var pares = numeros.Where(n => n % 2 == 0).ToList();
// [2, 4, 6, 8, 10]

var dobrados = numeros.Select(n => n * 2).ToList();
// [2, 4, 6, ...]

var soma = numeros.Sum();         // 55
var media = numeros.Average();    // 5.5
var maior = numeros.Max();        // 10
var quantosMaiorQue5 = numeros.Count(n => n > 5); // 5
```

LINQ merece um capítulo só dele. Por ora, saiba que **existe** e é poderoso.

---

## 5.8 Programa de exemplo

```csharp
List<string> tarefas = new List<string>();

while (true)
{
    Console.WriteLine("\n--- Lista de Tarefas ---");
    Console.WriteLine("1. Adicionar");
    Console.WriteLine("2. Listar");
    Console.WriteLine("3. Remover");
    Console.WriteLine("4. Sair");
    Console.Write("Opção: ");

    string opcao = Console.ReadLine();

    if (opcao == "1")
    {
        Console.Write("Nova tarefa: ");
        tarefas.Add(Console.ReadLine());
    }
    else if (opcao == "2")
    {
        for (int i = 0; i < tarefas.Count; i++)
        {
            Console.WriteLine($"{i + 1}. {tarefas[i]}");
        }
    }
    else if (opcao == "3")
    {
        Console.Write("Número da tarefa: ");
        int indice = int.Parse(Console.ReadLine()) - 1;
        if (indice >= 0 && indice < tarefas.Count)
        {
            tarefas.RemoveAt(indice);
        }
    }
    else if (opcao == "4")
    {
        break;
    }
}
```

---

## 5.9 Resumo do capítulo

- **Array** é tamanho fixo, simples e rápido.
- **`List<T>`** é dinâmica, é a mais usada no dia-a-dia.
- **`IEnumerable<T>`** é o "contrato mínimo" — só permite percorrer.
- **Receba `IEnumerable<T>`** em parâmetros, **retorne `List<T>`** quando faz sentido.
- LINQ funciona em qualquer `IEnumerable<T>` e é extremamente poderoso.

---

## 5.10 Exercícios

1. Crie uma `List<int>` com os números de 1 a 20 e imprima apenas os pares.
2. Faça um programa que peça nomes ao usuário até ele digitar "fim", e depois liste todos.
3. Crie um método `IEnumerable<int> Numeros()` que retorne os 10 primeiros números pares (use `yield return` se quiser desafio).
4. Use um `Dictionary<string, double>` para guardar o preço de produtos. Permita o usuário consultar pelo nome.
5. Dada uma `List<int>`, escreva um método que retorna **só os números maiores que a média**.

---

➡️ **Próximo capítulo:** [Capítulo 6 — Orientação a Objetos (Completo)](06-Orientacao-a-Objetos.md)
