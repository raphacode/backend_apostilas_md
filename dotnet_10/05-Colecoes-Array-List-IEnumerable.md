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

## 5.3 `List<T>` — a coleção mais usada

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

## 5.4 `IEnumerable<T>`

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

| Característica              | `List<T>`               | `IEnumerable<T>`                           |
| --------------------------- | ----------------------- | ------------------------------------------ |
| Pode adicionar/remover?     | ✅ Sim                   | ❌ Não                                      |
| Acesso por índice (`[i]`)?  | ✅ Sim                   | ❌ Não                                      |
| Sabe o tamanho rapidamente? | ✅ `Count` (instantâneo) | ⚠️ `Count()` percorre toda a coleção       |
| Carrega tudo na memória?    | ✅ Sim                   | ⚠️ Pode ser **lazy** (carrega sob demanda) |
| Para que serve?             | Manipular dados         | **Apenas ler / iterar**                    |

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

## 5.7 LINQ — consultar, filtrar e transformar coleções

**LINQ** significa *Language Integrated Query* (*consulta integrada à linguagem*). Na prática, é um conjunto de ferramentas do C# para fazer perguntas a uma coleção:

- “quais números são pares?”;
- “quais nomes começam com A?”;
- “coloque as notas da maior para a menor”;
- “qual é a média?”;
- “quantos itens atendem a uma condição?”.

LINQ **não é outra coleção** e não substitui `List<T>`. A `List<T>` guarda os dados; LINQ ajuda a **ler, filtrar, organizar, transformar ou calcular** usando esses dados. Ele funciona muito bem com `List<T>`, arrays e qualquer `IEnumerable<T>`.

> [!tip] Modelo mental
> Pense sempre em três partes: **fonte dos dados** -> **operação LINQ** -> **resultado**.  
> Exemplo: `numeros` -> `Where(...)` -> “somente os pares”.

### Preparação

Os métodos de LINQ ficam no namespace `System.Linq`:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
```

Nos exemplos deste capítulo será usada a **sintaxe de métodos**, porque ela deixa claro qual operação está sendo aplicada em cada etapa.

### Primeiro exemplo: filtrar com `Where`

```csharp
List<int> numeros = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

IEnumerable<int> pares = numeros.Where(numero => numero % 2 == 0);

Console.WriteLine(string.Join(", ", pares));
// Saída: 2, 4, 6, 8, 10
```

Leia esta linha com calma:

```csharp
numeros.Where(numero => numero % 2 == 0)
```

| Parte | Significado |
|---|---|
| `numeros` | A coleção que será consultada. |
| `Where(...)` | “Mantenha apenas os itens que passam na condição”. |
| `numero` | Nome temporário dado a **cada número**, um de cada vez. Poderia ser `n`, mas `numero` é mais claro no começo. |
| `=>` | Lê-se “para cada”. Ele separa o item recebido da regra aplicada a ele. |
| `numero % 2 == 0` | A condição: o resto da divisão por 2 é zero, então o número é par. |

Portanto, `numero => numero % 2 == 0` quer dizer: **“para cada número, mantenha-o se ele for par”**.

> [!important] `Where` não altera a lista original
> Depois de criar `pares`, a lista `numeros` ainda contém todos os números de `1` a `10`. `Where` cria uma **consulta de leitura**, não remove os ímpares da lista.

### Quando a consulta realmente acontece: `IEnumerable` e `ToList()`

O resultado de `Where`, `Select`, `OrderBy` e vários outros métodos costuma ser um `IEnumerable<T>`. Isso significa que a consulta pode esperar até o momento em que você a percorre.

```csharp
List<int> numeros = new List<int> { 1, 2, 3, 4, 5, 6 };

IEnumerable<int> pares = numeros.Where(numero => numero % 2 == 0);

numeros.Add(8);

Console.WriteLine(string.Join(", ", pares));
// Saída: 2, 4, 6, 8
```

O `8` aparece porque `pares` ainda é uma consulta. Ao pedir os resultados, LINQ lê a lista atualizada.

Quando você precisa guardar o resultado em uma lista independente, use `ToList()`:

```csharp
List<int> paresFixos = numeros
    .Where(numero => numero % 2 == 0)
    .ToList();

numeros.Add(10);

Console.WriteLine(string.Join(", ", paresFixos));
// Saída: 2, 4, 6, 8
// O 10 não entra: paresFixos já foi criado.
```

Use `ToList()` quando precisar, por exemplo, acessar por índice, adicionar/remover itens no resultado ou garantir que a consulta seja feita uma única vez naquele momento.

### Transformar dados com `Select`

Enquanto `Where` decide **quais itens ficam**, `Select` decide **como cada item será transformado**.

```csharp
List<int> numeros = new List<int> { 1, 2, 3, 4, 5 };

IEnumerable<int> dobrados = numeros.Select(numero => numero * 2);
IEnumerable<string> mensagens = numeros.Select(numero => $"Número: {numero}");

Console.WriteLine(string.Join(", ", dobrados));
// Saída: 2, 4, 6, 8, 10

Console.WriteLine(string.Join(" | ", mensagens));
// Saída: Número: 1 | Número: 2 | Número: 3 | Número: 4 | Número: 5
```

| Método | Pergunta que ele responde | Exemplo |
|---|---|---|
| `Where` | “Quais itens devem ficar?” | `Where(n => n > 5)` |
| `Select` | “Em que cada item deve se transformar?” | `Select(n => n * 2)` |

### Encadeando operações: uma etapa depois da outra

As operações LINQ podem ser encadeadas. Leia de cima para baixo: primeiro filtra, depois ordena, depois limita e por último transforma.

```csharp
List<int> numeros = new List<int> { 3, 10, 1, 8, 6, 5, 2, 4, 9, 7 };

List<int> tresMaioresParesAoQuadrado = numeros
    .Where(numero => numero % 2 == 0)          // 10, 8, 6, 2, 4
    .OrderByDescending(numero => numero)       // 10, 8, 6, 4, 2
    .Take(3)                                    // 10, 8, 6
    .Select(numero => numero * numero)         // 100, 64, 36
    .ToList();

Console.WriteLine(string.Join(", ", tresMaioresParesAoQuadrado));
// Saída: 100, 64, 36
```

> [!tip] Como ler um encadeamento
> Comece pela coleção antes do primeiro ponto. Depois acompanhe cada linha: **filtre** -> **ordene** -> **pegue uma parte** -> **transforme** -> **guarde**, se necessário.

### Métodos LINQ mais úteis no começo

| Método | O que faz | Exemplo | Resultado |
|---|---|---|---|
| `Where` | Filtra itens por uma condição. | `numeros.Where(n => n >= 7)` | Os números `7` ou maiores. |
| `Select` | Transforma cada item. | `numeros.Select(n => n * 2)` | Cada número dobrado. |
| `OrderBy` | Ordena do menor para o maior. | `numeros.OrderBy(n => n)` | Ordem crescente. |
| `OrderByDescending` | Ordena do maior para o menor. | `numeros.OrderByDescending(n => n)` | Ordem decrescente. |
| `Take` | Pega os primeiros itens. | `numeros.Take(3)` | Os três primeiros. |
| `Skip` | Pula os primeiros itens. | `numeros.Skip(3)` | Itens depois dos três primeiros. |
| `Distinct` | Remove valores repetidos no resultado. | `numeros.Distinct()` | Cada valor uma vez. |
| `Any` | Verifica se existe ao menos um item. | `numeros.Any(n => n < 0)` | `true` ou `false`. |
| `Count` | Conta itens; pode receber condição. | `numeros.Count(n => n > 5)` | Um número inteiro. |
| `Sum`, `Average`, `Min`, `Max` | Calculam um único valor numérico. | `numeros.Average()` | Uma média. |
| `FirstOrDefault` | Pega o primeiro item ou o valor padrão se não houver. | `numeros.FirstOrDefault(n => n > 50)` | `0` para `int` quando não encontra. |
| `ToList` | Executa e guarda o resultado em uma `List<T>`. | `consulta.ToList()` | Uma lista independente. |

### Calcular não é filtrar

Métodos como `Where` e `Select` devolvem uma sequência de itens. Métodos como `Count`, `Sum` e `Average` devolvem **um único valor** e precisam percorrer a consulta para calculá-lo.

```csharp
List<double> notas = new List<double> { 4.5, 6.0, 7.5, 8.0, 10.0 };

int quantidadeAprovados = notas.Count(nota => nota >= 6.0);
double media = notas.Average();
double maiorNota = notas.Max();
bool existeNotaDez = notas.Any(nota => nota == 10.0);

Console.WriteLine($"Aprovados: {quantidadeAprovados}"); // 4
Console.WriteLine($"Média: {media}");                   // 7,2 ou 7.2, conforme a cultura
Console.WriteLine($"Maior nota: {maiorNota}");           // 10
Console.WriteLine($"Existe nota 10? {existeNotaDez}");   // True
```

### Programa completo: relatório simples de notas

Este exemplo junta filtro, ordenação, cálculo e verificação. Ele pode ser copiado para um projeto Console.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

List<double> notas = new List<double> { 4.5, 6.0, 7.5, 8.0, 10.0, 5.0 };

List<double> aprovados = notas
    .Where(nota => nota >= 6.0)
    .OrderByDescending(nota => nota)
    .ToList();

double mediaDaTurma = notas.Average();
int quantidadeReprovados = notas.Count(nota => nota < 6.0);
double primeiraNotaPerfeita = notas.FirstOrDefault(nota => nota == 10.0);

Console.WriteLine("--- Relatório de Notas ---");
Console.WriteLine($"Notas registradas: {string.Join(", ", notas)}");
Console.WriteLine($"Aprovados em ordem decrescente: {string.Join(", ", aprovados)}");
Console.WriteLine($"Média da turma: {mediaDaTurma:F2}");
Console.WriteLine($"Quantidade abaixo de 6: {quantidadeReprovados}");
Console.WriteLine($"Existe nota 10? {notas.Any(nota => nota == 10.0)}");
Console.WriteLine($"Primeira nota 10 encontrada: {primeiraNotaPerfeita}");
```

Resultado esperado:

```txt
--- Relatório de Notas ---
Notas registradas: 4.5, 6, 7.5, 8, 10, 5
Aprovados em ordem decrescente: 10, 8, 7.5, 6
Média da turma: 6,83 (ou 6.83, conforme a configuração do computador)
Quantidade abaixo de 6: 2
Existe nota 10? True
Primeira nota 10 encontrada: 10
```

### Cuidados importantes

1. **`Where` não retorna uma `List<T>` automaticamente.** Se precisar usar `Add`, `Remove` ou índice no resultado, termine com `ToList()`.
2. **`First()` pode lançar erro** quando não encontra nenhum item. Para começar com mais segurança, prefira `FirstOrDefault()` e confira o valor devolvido. Para `int`, o padrão é `0`; se `0` também puder ser um resultado válido, use `Any(...)` para saber se o item realmente existe.
3. **Uma consulta pode ser executada mais de uma vez.** Se a fonte muda entre duas leituras, o resultado também pode mudar. Use `ToList()` quando quiser congelar aquele resultado.
4. **LINQ não é obrigatório em todo `foreach`.** Se você precisa alterar cada item, guardar várias decisões ou o laço fica mais claro, `foreach` continua sendo uma excelente escolha.
5. **Dê nomes claros às variáveis da expressão lambda.** `nota => nota >= 6` ensina mais do que `n => n >= 6` quando você ainda está aprendendo.

### Exercite antes de seguir

1. Crie uma lista com os números de `1` a `20` e use `Where` para mostrar apenas os múltiplos de `3`.
2. Use `Select` para criar outra sequência com o quadrado de cada número de `1` a `10`.
3. Dada uma lista de notas, mostre apenas as notas maiores ou iguais a `6`, em ordem decrescente.
4. Em uma lista com valores repetidos, use `Distinct` e depois `OrderBy` para mostrar cada valor uma única vez, em ordem crescente.
5. Calcule a média e use `Where` para encontrar as notas maiores que a média.

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

➡️ **Próximo capítulo:**[Capítulo 6 — Orientação a Objetos (Completo)](3-Apostilas/backend/01_csharp_dotnet/dotnet_10/06-Orientacao-a-Objetos.md))

---
[[1-index|Voltar para o index]]
