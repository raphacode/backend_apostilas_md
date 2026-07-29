# Capítulo 6 — Orientação a Objetos (Completo)

> "Programar é modelar a realidade." Orientação a Objetos (OO) é a forma mais usada no mundo de **organizar essa modelagem**. Este capítulo é o coração da apostila — leia com calma, releia se precisar.

---

## 6.1 O que é Orientação a Objetos?

Imagine que você quer criar um sistema para uma **escola**. Nesse sistema existem:

- **Alunos** (com nome, idade, matrícula).
- **Professores** (com nome, matéria que leciona).
- **Disciplinas** (com nome, carga horária).

Cada um deles tem **dados** (atributos) e **comportamentos** (ações). Aluno **estuda**, professor **dá aula**, disciplina **gera notas**.

A **Orientação a Objetos** é uma forma de programar onde a gente **modela esses elementos do mundo real como "objetos"** dentro do código, com seus dados e comportamentos juntos, em **classes**.

### Vantagens da OO

- **Organização**: cada coisa tem seu lugar.
- **Reutilização**: você cria uma classe e usa em vários lugares.
- **Facilidade de manutenção**: mudou uma regra de negócio? Você sabe exatamente onde mexer.
- **Modela o mundo real** de forma intuitiva.

---

## 6.2 Classes e Objetos

### Classe = molde / planta

Uma **classe** é uma **definição abstrata**. É como a **planta de uma casa**: descreve o que tem na casa, mas não é a casa em si.

### Objeto = instância / casa construída

Um **objeto** é uma **instância** dessa classe. É a **casa real**, construída a partir da planta.

### Exemplo

```csharp
public class Aluno
{
    public string Nome;
    public int Idade;

    public void Estudar()
    {
        Console.WriteLine($"{Nome} está estudando.");
    }
}
```

Isso é **a classe** `Aluno`. Não há nenhum aluno ainda — só uma definição.

Agora vamos **criar um objeto** (instanciar):

```csharp
Aluno maria = new Aluno();
maria.Nome = "Maria";
maria.Idade = 20;
maria.Estudar(); // "Maria está estudando."

Aluno joao = new Aluno();
joao.Nome = "João";
joao.Idade = 22;
joao.Estudar(); // "João está estudando."
```

`maria` e `joao` são **objetos diferentes**, criados a partir da **mesma classe**.

> A palavra **`new`** é o que cria um novo objeto na memória.

---

## 6.3 Atributos (campos e propriedades)

**Atributos** são os **dados** que um objeto guarda.

### Campos (fields)

Forma simples e direta:

```csharp
public class Aluno
{
    public string Nome;
    public int Idade;
}
```

### Propriedades (properties) — o jeito "C#"

Em C#, é **muito mais comum** usar **propriedades** em vez de campos públicos. Propriedades parecem campos, mas permitem **controlar leitura e escrita**:

```csharp
public class Aluno
{
    public string Nome { get; set; }
    public int Idade { get; set; }
}
```

`{ get; set; }` é uma propriedade automática:
- `get` = pode ler.
- `set` = pode escrever.

Você usa **igual a um campo**:

```csharp
Aluno maria = new Aluno();
maria.Nome = "Maria";        // chama o set
Console.WriteLine(maria.Nome); // chama o get
```

### Por que propriedades em vez de campos?

Porque você pode **adicionar lógica** depois, sem quebrar quem usa a classe:

```csharp
public class Aluno
{
    private string _nome;

    public string Nome
    {
        get { return _nome; }
        set
        {
            if (string.IsNullOrWhiteSpace(value))
                throw new ArgumentException("Nome não pode ser vazio");
            _nome = value;
        }
    }
}
```

Agora a classe **valida** o nome ao definir.

### Propriedade só de leitura

```csharp
public string Codigo { get; }
```

Só pode ser definida no construtor. Depois, ninguém modifica.

---

## 6.4 Métodos

**Métodos** são os **comportamentos** do objeto.

```csharp
public class Aluno
{
    public string Nome { get; set; }
    public List<double> Notas { get; set; } = new List<double>();

    public double CalcularMedia()
    {
        if (Notas.Count == 0) return 0;

        double soma = 0;
        foreach (double nota in Notas)
            soma += nota;

        return soma / Notas.Count;
    }

    public bool EstaAprovado()
    {
        return CalcularMedia() >= 7;
    }
}
```

Uso:

```csharp
Aluno aluno = new Aluno { Nome = "Maria" };
aluno.Notas.Add(8);
aluno.Notas.Add(7.5);
aluno.Notas.Add(9);

Console.WriteLine(aluno.CalcularMedia());   // 8.16
Console.WriteLine(aluno.EstaAprovado());    // true
```

> A sintaxe `new Aluno { Nome = "Maria" }` é o **inicializador de objeto** — atribui propriedades já na criação.

### `this`

Dentro de um método ou propriedade, `this` significa "**este objeto**". Útil quando há ambiguidade:

```csharp
public class Aluno
{
    public string Nome { get; set; }

    public void DefinirNome(string nome)
    {
        this.Nome = nome; // o "this." deixa claro qual é qual
    }
}
```

---

## 6.5 Construtores

**Construtor** é um método especial que executa **quando o objeto é criado** (no `new`). Serve para inicializar o objeto.

```csharp
public class Aluno
{
    public string Nome { get; set; }
    public int Idade { get; set; }

    // Construtor
    public Aluno(string nome, int idade)
    {
        Nome = nome;
        Idade = idade;
    }
}

// Uso:
Aluno maria = new Aluno("Maria", 20);
```

### Regras de construtores

- Tem o **mesmo nome da classe**.
- **Não tem tipo de retorno** (nem `void`).
- Pode ter **vários** (sobrecarga):

```csharp
public Aluno() { }                            // sem parâmetros
public Aluno(string nome) { Nome = nome; }    // só nome
public Aluno(string nome, int idade)          // ambos
{
    Nome = nome;
    Idade = idade;
}
```

Se você **não declarar nenhum** construtor, o C# cria automaticamente um construtor vazio.

---

## 6.6 Encapsulamento

**Encapsulamento** é o pilar mais importante da OO. Significa: **esconder os detalhes internos** e **expor apenas o necessário**.

### Analogia

Pense num **carro**. Você usa o **volante**, o **acelerador** e o **freio** (a "interface" do carro). Você **não precisa saber** como funciona o motor, a injeção eletrônica, o ABS. Tudo isso está **encapsulado**.

Se a Volkswagen mudar o motor amanhã, **você dirige igual**. Porque o que está exposto (interface) não mudou.

### Em código

❌ **Sem encapsulamento** (ruim):

```csharp
public class ContaBancaria
{
    public double Saldo;  // qualquer um pode mexer livremente
}

ContaBancaria conta = new ContaBancaria();
conta.Saldo = -1_000_000; // OPA! Saldo negativo? Que absurdo.
```

✅ **Com encapsulamento** (bom):

```csharp
public class ContaBancaria
{
    public double Saldo { get; private set; }  // ler todo mundo, escrever só a classe

    public void Depositar(double valor)
    {
        if (valor <= 0)
            throw new ArgumentException("Valor deve ser positivo");
        Saldo += valor;
    }

    public void Sacar(double valor)
    {
        if (valor <= 0)
            throw new ArgumentException("Valor deve ser positivo");
        if (valor > Saldo)
            throw new InvalidOperationException("Saldo insuficiente");
        Saldo -= valor;
    }
}
```

Agora **ninguém consegue colocar saldo inválido**. A classe controla seu próprio estado.

> **Regra de ouro:** atributos quase sempre `private` (ou `private set`). Métodos públicos são a "interface" da classe.

---

## 6.7 Herança

**Herança** permite criar uma classe que **herda** características de outra.

### Quando faz sentido?

Quando uma classe é **um tipo específico** de outra. Pense em "**é um/uma**":

- Cachorro **é um** Animal.
- Carro **é um** Veículo.
- Gerente **é um** Funcionário.

### Sintaxe

```csharp
public class Animal
{
    public string Nome { get; set; }

    public void Comer()
    {
        Console.WriteLine($"{Nome} está comendo.");
    }
}

public class Cachorro : Animal   // Cachorro herda de Animal
{
    public void Latir()
    {
        Console.WriteLine($"{Nome} está latindo!");
    }
}
```

Uso:

```csharp
Cachorro rex = new Cachorro();
rex.Nome = "Rex";  // herdado de Animal
rex.Comer();       // herdado
rex.Latir();       // próprio
```

### Vocabulário

- **Classe base** (ou **superclasse**): a "pai" — `Animal`.
- **Classe derivada** (ou **subclasse**): a "filha" — `Cachorro`.

### Construtor com herança

A classe filha precisa "**chamar**" o construtor do pai com `base`:

```csharp
public class Animal
{
    public string Nome { get; set; }

    public Animal(string nome)
    {
        Nome = nome;
    }
}

public class Cachorro : Animal
{
    public string Raca { get; set; }

    public Cachorro(string nome, string raca) : base(nome)
    {
        Raca = raca;
    }
}
```

### Cuidado com herança

Herança é **poderosa, mas perigosa**. Use **só quando for "é um/uma" de verdade**. Se for só "tem um/uma", use **composição** (um atributo do tipo da outra classe).

> **Regra prática:** prefira composição a herança.

---

## 6.8 Polimorfismo

**Polimorfismo** = "muitas formas". Significa que **o mesmo método pode se comportar de jeitos diferentes** em classes diferentes.

### Sobrescrita de métodos (`virtual` / `override`)

Na classe pai, marque o método como `virtual`. Na filha, use `override`:

```csharp
public class Animal
{
    public string Nome { get; set; }

    public virtual void EmitirSom()
    {
        Console.WriteLine("Som genérico");
    }
}

public class Cachorro : Animal
{
    public override void EmitirSom()
    {
        Console.WriteLine("Au au!");
    }
}

public class Gato : Animal
{
    public override void EmitirSom()
    {
        Console.WriteLine("Miau!");
    }
}
```

### Onde está a mágica?

```csharp
List<Animal> animais = new List<Animal>
{
    new Cachorro { Nome = "Rex" },
    new Gato { Nome = "Mimi" },
    new Cachorro { Nome = "Bob" }
};

foreach (Animal animal in animais)
{
    animal.EmitirSom();
    // Rex: Au au!
    // Mimi: Miau!
    // Bob: Au au!
}
```

A variável é do tipo `Animal`, mas C# chama o método **certo** de acordo com o objeto real. **Isso é polimorfismo.**

### Por que isso é genial?

Você pode **adicionar um novo animal** (`Vaca`) sem mexer no código que percorre a lista. **O código fica aberto para extensão e fechado para modificação** (princípio aberto/fechado).

---

## 6.9 Abstração

**Abstração** é o ato de **focar no essencial e ignorar os detalhes**.

### Em código

Quando você cria uma classe `Pagamento` com método `Pagar()`, não importa **como** ele paga (cartão? Pix? boleto?). O usuário da classe só se importa **que pague**.

A abstração se manifesta em C# de duas formas principais:

- **Classes abstratas** — base que não pode ser instanciada.
- **Interfaces** — contratos puros.

Vamos detalhar ambas no **Capítulo 8**, com calma.

### Em uma frase

> Abstração = expor **o que** o objeto faz, não **como** ele faz.

---

## 6.10 Os 4 pilares da OO (revisão visual)

| Pilar | Em uma frase |
|---|---|
| **Encapsulamento** | Esconder os detalhes; expor o necessário. |
| **Herança** | Reutilizar código entre classes "parentes". |
| **Polimorfismo** | Mesmo método, comportamentos diferentes. |
| **Abstração** | Focar no essencial; esconder o complexo. |

Decore esses 4. Você vai ouvir falar deles a vida inteira.

---

## 6.11 Exemplo completo

```csharp
public class Funcionario
{
    public string Nome { get; set; }
    public double SalarioBase { get; set; }

    public Funcionario(string nome, double salario)
    {
        Nome = nome;
        SalarioBase = salario;
    }

    public virtual double CalcularSalario()
    {
        return SalarioBase;
    }
}

public class Gerente : Funcionario
{
    public double Bonus { get; set; }

    public Gerente(string nome, double salario, double bonus)
        : base(nome, salario)
    {
        Bonus = bonus;
    }

    public override double CalcularSalario()
    {
        return SalarioBase + Bonus;
    }
}

public class Estagiario : Funcionario
{
    public Estagiario(string nome, double salario)
        : base(nome, salario) { }

    public override double CalcularSalario()
    {
        return SalarioBase * 0.5; // estagiário recebe metade
    }
}
```

Uso:

```csharp
List<Funcionario> equipe = new List<Funcionario>
{
    new Gerente("Maria", 5000, 2000),
    new Funcionario("João", 3000),
    new Estagiario("Ana", 2000)
};

foreach (Funcionario f in equipe)
{
    Console.WriteLine($"{f.Nome}: R$ {f.CalcularSalario():F2}");
}
// Maria: R$ 7000.00
// João: R$ 3000.00
// Ana: R$ 1000.00
```

---

## 6.12 Resumo do capítulo

- **Classe** é o molde; **objeto** é a instância concreta.
- **Atributos** são dados; **métodos** são comportamentos.
- **Propriedades** são a forma C# de expor atributos com controle.
- **Construtor** inicializa o objeto.
- **Encapsulamento** protege o estado interno.
- **Herança** permite reaproveitar código entre classes relacionadas.
- **Polimorfismo** permite que o mesmo método se comporte de jeitos diferentes.
- **Abstração** foca no essencial.

---

## 6.13 Exercícios

1. Crie uma classe `Pessoa` com `Nome`, `Idade` e um método `Apresentar()`.
2. Crie uma classe `ContaBancaria` com encapsulamento adequado: `Saldo` privado, métodos `Depositar`, `Sacar`, `Transferir`.
3. Crie uma hierarquia: `Veiculo` → `Carro`, `Moto`, `Caminhao`. Cada um com método `Acelerar()` polimórfico.
4. Crie uma classe `Produto` com `Nome` e `Preco`, e uma classe `ProdutoComDesconto : Produto` que sobrescreve a forma de calcular o preço.
5. Modele uma `Turma` que tem uma `List<Aluno>`. Crie métodos `AdicionarAluno`, `MediaDaTurma`, `Aprovados`.

---

➡️ **Próximo capítulo:** [Capítulo 7 — Modificadores de Acesso e `static`](07-Modificadores-e-Static.md)
