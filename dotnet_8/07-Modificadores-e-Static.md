# Capítulo 7 — Modificadores de Acesso e `static`

> Quem pode ver o quê? Quem precisa de instância? Este capítulo responde essas perguntas que **todo iniciante tem** ao começar OO em C#.

---

## 7.1 Modificadores de acesso — visão geral

Modificadores de acesso são **palavras-chave** que controlam **quem pode ver e usar** classes, atributos e métodos.

Os principais em C#:

| Modificador | Visibilidade |
|---|---|
| `public` | **Qualquer um**, em qualquer projeto. |
| `private` | **Só dentro da própria classe**. |
| `protected` | A própria classe **e suas subclasses**. |
| `internal` | **Qualquer um, mas só dentro do mesmo projeto/assembly**. |
| `protected internal` | Subclasses **ou** mesmo projeto. |
| `private protected` | Subclasses **dentro do mesmo projeto**. |

Os **4 primeiros** são os que você usa 99% do tempo.

---

## 7.2 `public` — visível em qualquer lugar

```csharp
public class Usuario
{
    public string Nome { get; set; }

    public void Saudar()
    {
        Console.WriteLine($"Olá, {Nome}");
    }
}
```

Qualquer código, em qualquer projeto que **referencie** o seu, consegue:

```csharp
Usuario u = new Usuario();
u.Nome = "Maria";
u.Saudar();
```

> **Use `public`** para o que você quer **expor como interface** da classe.

---

## 7.3 `private` — visível só dentro da classe

```csharp
public class ContaBancaria
{
    private double _saldo;          // só essa classe enxerga
    private string _senha;

    public void Depositar(double valor)
    {
        _saldo += valor; // ok, estamos na mesma classe
    }
}
```

De fora:

```csharp
ContaBancaria c = new ContaBancaria();
// c._saldo = 1000; // ERRO de compilação!
```

> **Use `private`** para **detalhes internos** que ninguém de fora deveria mexer. **A maioria dos atributos deve ser `private`**.

---

## 7.4 `internal` — visível só no mesmo projeto

```csharp
internal class HelperDeCalculo
{
    internal int Somar(int a, int b) => a + b;
}
```

`internal` é o **default** para classes em C# se você não especificar nada.

### Quando usar?

Quando a classe é um **detalhe de implementação interno do seu projeto** e você **não quer** que outros projetos (que dependam do seu) vejam.

Exemplo: você está fazendo uma biblioteca. `public` = parte da API. `internal` = uso interno seu.

---

## 7.5 `protected` — só na classe e nas filhas

```csharp
public class Animal
{
    protected string SomCaracteristico { get; set; }

    protected void Respirar()
    {
        Console.WriteLine("Respirando...");
    }
}

public class Cachorro : Animal
{
    public Cachorro()
    {
        SomCaracteristico = "Au au"; // ok, herdou
        Respirar();                  // ok, herdou
    }
}

// De fora:
Cachorro rex = new Cachorro();
// rex.Respirar();  // ERRO! protected não é visível externamente
```

> **Use `protected`** para coisas que as **subclasses precisam acessar**, mas que não devem ser expostas ao mundo todo.

---

## 7.6 Comparativo prático

```csharp
public class Exemplo
{
    public string A;       // mundo todo
    internal string B;     // mesmo projeto
    protected string C;    // classe + subclasses
    private string D;      // só esta classe
}
```

| De onde estou olhando? | Vejo `A`? | `B`? | `C`? | `D`? |
|---|:-:|:-:|:-:|:-:|
| Mesma classe | ✅ | ✅ | ✅ | ✅ |
| Subclasse no mesmo projeto | ✅ | ✅ | ✅ | ❌ |
| Subclasse em outro projeto | ✅ | ❌ | ✅ | ❌ |
| Outra classe no mesmo projeto | ✅ | ✅ | ❌ | ❌ |
| Outra classe em outro projeto | ✅ | ❌ | ❌ | ❌ |

---

## 7.7 Boas práticas de visibilidade

1. **Comece sempre o mais restritivo possível.** Vá abrindo só quando precisar.
2. **Atributos: quase sempre `private`.** Exponha por propriedades.
3. **Métodos auxiliares internos: `private`.**
4. **A "interface" da classe (o que ela oferece): `public`.**
5. **Helpers internos do projeto: `internal`.**

---

## 7.8 `static` — pertence à classe, não ao objeto

`static` é um modificador que muda fundamentalmente o comportamento de uma classe ou membro.

### O conceito-chave

> **`static` significa que o membro pertence à CLASSE, não a um OBJETO.**

Sem `static`:
- Você precisa **criar um objeto** (`new`) para usar.
- Cada objeto tem sua **própria cópia** dos atributos.

Com `static`:
- Você usa **direto pela classe**, sem `new`.
- Existe **uma única cópia** compartilhada.

### Exemplo prático

```csharp
public class Calculadora
{
    public static double Somar(double a, double b)
    {
        return a + b;
    }

    public static double Pi { get; } = 3.14159;
}

// Uso:
double resultado = Calculadora.Somar(3, 5);   // sem new!
double pi = Calculadora.Pi;
```

Compare com algo **não estático**:

```csharp
public class Calculadora
{
    public double Somar(double a, double b) => a + b;
}

Calculadora c = new Calculadora();   // precisa criar objeto
double resultado = c.Somar(3, 5);
```

### Atributos estáticos compartilham valor

```csharp
public class Contador
{
    public static int Total = 0;
}

Contador.Total++;
Contador.Total++;
Contador.Total++;
Console.WriteLine(Contador.Total); // 3
```

**Não há "instância"** — o valor é único, da classe.

---

## 7.9 Classes `static`

Você pode marcar a **classe inteira** como `static`:

```csharp
public static class Utilidades
{
    public static string Maiuscula(string s) => s.ToUpper();
    public static int Dobrar(int n) => n * 2;
}
```

### Regras das classes `static`

> **Pontos de ouro do user (memorize):**
> - Uma classe `static` **não precisa ser instanciada** porque já fica **carregada em nível de classe**.
> - **Todos** os métodos e atributos de uma classe `static` **também precisam ser `static`** (o compilador exige).

```csharp
public static class Utilidades
{
    public string Nome;        // ❌ ERRO
    public void Fazer() { }    // ❌ ERRO
}
```

Você **não pode** criar `new Utilidades()`. A classe é "estática" — só serve para guardar funções e dados globais.

### Quando usar uma classe `static`?

- Para **utilitários puros** (funções que só dependem de seus parâmetros).
- Para **constantes globais**.
- Quando **não há estado** a manter.

> Exemplo do .NET: a classe `Math` (`Math.Sqrt(9)`, `Math.Max(a, b)`) é `static`. Faz sentido — não há um "objeto Math".

### Cuidados com `static`

- **Estado estático mutável é perigoso**: como é compartilhado, pode causar bugs em código concorrente.
- **Difícil de testar**: não dá para "trocar" um método estático em testes.
- **Acopla seu código**: quem usa fica preso àquela classe específica.

> **Regra prática:** use `static` para **utilitários sem estado**. Para regras de negócio, prefira classes normais com instâncias.

---

## 7.10 `static` em métodos individuais

Você pode ter métodos `static` dentro de uma classe **normal**:

```csharp
public class Aluno
{
    public string Nome { get; set; }

    // Método de instância: precisa de objeto
    public void Estudar() { /* ... */ }

    // Método estático: chamado pela classe
    public static Aluno Criar(string nome)
    {
        return new Aluno { Nome = nome };
    }
}

// Uso:
Aluno a = Aluno.Criar("Maria");  // método estático
a.Estudar();                      // método de instância
```

Métodos estáticos **não conseguem acessar** atributos/métodos de instância (porque não há instância).

---

## 7.11 `const` vs `static readonly`

Duas formas de declarar valores que não mudam:

```csharp
public class Configuracao
{
    public const double Pi = 3.14159;
    public static readonly DateTime IniciadoEm = DateTime.Now;
}
```

| | `const` | `static readonly` |
|---|---|---|
| Quando o valor é definido | **Em tempo de compilação** | **Em tempo de execução** |
| Pode usar valores dinâmicos? | ❌ Não | ✅ Sim (ex: `DateTime.Now`) |
| Uso típico | Números, strings literais | Valores calculados |

---

## 7.12 Resumo do capítulo

- `public`: visível em todo lugar.
- `private`: só dentro da classe.
- `internal`: só dentro do projeto/assembly.
- `protected`: na classe e nas subclasses.
- `static` significa "**pertence à classe, não à instância**".
- Classes `static` **só têm membros `static`** e **não podem ser instanciadas**.
- Use `static` para utilitários **sem estado**; evite para lógica de negócio.

---

## 7.13 Exercícios

1. Crie uma classe `MatematicaUtil` (estática) com métodos `Quadrado`, `Cubo` e `Fatorial`.
2. Crie uma classe `Usuario` com `Senha` privada e métodos públicos `DefinirSenha` e `ValidarSenha`.
3. Crie uma classe `Animal` com atributo `protected` `Especie`. Mostre que uma classe filha consegue acessar, mas código externo não.
4. Crie uma classe `Contador` com um atributo estático `TotalCriados` que é incrementado a cada `new Contador()`. Use o construtor.
5. Tente criar uma classe `static` com um método de instância. Veja a mensagem de erro do compilador. Conserte.

---

➡️ **Próximo capítulo:** [Capítulo 8 — Interfaces e Classes Abstratas](08-Interfaces-e-Classes-Abstratas.md)
