# Capítulo 8 — Interfaces e Classes Abstratas

> Os dois recursos mais usados em código profissional para **abstração**. Saber qual usar em cada situação separa o iniciante do programador maduro.

---

## 8.1 O que é uma Interface?

> **Uma interface é um contrato.** Ela define **o que** uma classe deve fazer, **sem dizer como**.

Pense em uma **tomada elétrica**. A tomada é um "contrato": qualquer aparelho que tenha o **plugue compatível** funciona. A tomada não se importa se é uma TV, um liquidificador ou um carregador — só importa que respeite o contrato.

### Sintaxe

```csharp
public interface IFormaGeometrica
{
    double CalcularArea();
    double CalcularPerimetro();
}
```

> **Convenção em C#:** nomes de interface começam com **`I` maiúsculo** (`IUsuario`, `IServico`, `IRepositorio`).

### Implementando uma interface

Quando uma classe **implementa** uma interface, ela é **obrigada** a fornecer todos os métodos do contrato:

```csharp
public class Retangulo : IFormaGeometrica
{
    public double Largura { get; set; }
    public double Altura { get; set; }

    public double CalcularArea() => Largura * Altura;
    public double CalcularPerimetro() => 2 * (Largura + Altura);
}

public class Circulo : IFormaGeometrica
{
    public double Raio { get; set; }

    public double CalcularArea() => Math.PI * Raio * Raio;
    public double CalcularPerimetro() => 2 * Math.PI * Raio;
}
```

### Por que isso é poderoso?

```csharp
List<IFormaGeometrica> formas = new List<IFormaGeometrica>
{
    new Retangulo { Largura = 3, Altura = 4 },
    new Circulo { Raio = 5 }
};

foreach (IFormaGeometrica f in formas)
{
    Console.WriteLine($"Área: {f.CalcularArea():F2}");
}
```

A lista **não sabe nem se importa** se é retângulo ou círculo. Ela trabalha com o **contrato**.

> **Em uma frase:** interfaces permitem **trocar implementações** sem mudar o código que as usa.

---

## 8.2 Características das interfaces

- **Não podem ser instanciadas**: `new IFormaGeometrica()` é erro.
- Tradicionalmente **só declaram membros, sem implementação**. (A partir do C# 8 é possível ter implementação padrão, mas é raro usar.)
- Uma classe pode implementar **várias interfaces** (separadas por vírgula):

```csharp
public class Pato : INadador, IVoador, ICantor
{
    public void Nadar() { }
    public void Voar() { }
    public void Cantar() { }
}
```

> **C# não permite herança múltipla** de classes (uma classe só herda de UMA). Mas **interfaces múltiplas — sim, sem limite**.

---

## 8.3 Casos de uso reais de interfaces

### 1. Definir um serviço (muito comum em backend)

```csharp
public interface IEmailService
{
    void Enviar(string para, string assunto, string corpo);
}

public class EmailServiceSendGrid : IEmailService
{
    public void Enviar(string para, string assunto, string corpo)
    {
        // chama a API do SendGrid
    }
}

public class EmailServiceFake : IEmailService
{
    public void Enviar(string para, string assunto, string corpo)
    {
        Console.WriteLine($"FAKE: Email para {para}");
    }
}
```

Em **produção** você usa `EmailServiceSendGrid`. Em **testes**, usa `EmailServiceFake`. **Sem mudar uma linha do código que envia o e-mail.**

### 2. Permitir várias implementações

```csharp
public interface IRepositorioUsuario
{
    Usuario Buscar(int id);
    void Salvar(Usuario u);
}

// Pode haver: RepositorioUsuarioSql, RepositorioUsuarioMongoDb, RepositorioUsuarioMemoria...
```

---

## 8.4 O que é uma Classe Abstrata?

> **Uma classe abstrata é um "modelo base" para outras classes.** Ela pode ter código pronto, mas **não pode ser instanciada diretamente**.

### Pontos importantes (memorize)

> **Pontos de ouro:**
> - Uma classe abstrata é um **modelo/base** para classes concretas, mas **não pode ser instanciada**.
> - Ela pode ter **métodos com implementação E métodos sem implementação**.

### Sintaxe

```csharp
public abstract class Animal
{
    public string Nome { get; set; }

    // Método com implementação (concreto)
    public void Respirar()
    {
        Console.WriteLine($"{Nome} está respirando.");
    }

    // Método SEM implementação (abstrato) - filhas obrigadas a implementar
    public abstract void EmitirSom();
}
```

### Tentando instanciar

```csharp
Animal a = new Animal(); // ❌ ERRO! Não pode.
```

### Implementando classes filhas

```csharp
public class Cachorro : Animal
{
    public override void EmitirSom()
    {
        Console.WriteLine("Au au!");
    }
}

public class Vaca : Animal
{
    public override void EmitirSom()
    {
        Console.WriteLine("Muuu!");
    }
}
```

A filha **deve** sobrescrever (`override`) os métodos abstratos. Ela **herda** os concretos.

### Uso

```csharp
Cachorro rex = new Cachorro { Nome = "Rex" };
rex.Respirar();    // herdado
rex.EmitirSom();   // próprio

Animal a = rex;    // ok, polimorfismo
a.EmitirSom();     // chama o do cachorro
```

---

## 8.5 Diferenças: Interface vs. Classe Abstrata

| Característica | Interface | Classe Abstrata |
|---|---|---|
| Tem implementação? | **Geralmente não** (só assinaturas) | **Pode ter os dois**: métodos abstratos e concretos |
| Pode ter atributos com estado? | **Não** | **Sim** |
| Construtor? | **Não** | **Sim** |
| Pode ser instanciada? | ❌ Não | ❌ Não |
| Quantas uma classe pode "ter"? | **Várias** (`A : I1, I2, I3`) | **Apenas uma** (herança simples) |
| Modificadores de acesso nos membros | Sempre `public` (implicitamente) | Qualquer um |
| Quando usar | Definir **comportamentos/contratos** | Compartilhar **código base** entre classes irmãs |

---

## 8.6 Quando usar Interface vs. Classe Abstrata

### Use **Interface** quando:

- Você quer definir **um contrato puro**: "se você é X, você deve fazer A, B e C".
- As classes que vão implementar **não têm relação hierárquica** entre si.
- Você precisa que uma classe **implemente vários comportamentos** (várias interfaces).
- Em **arquitetura de backend** (controllers, serviços, repositórios), interfaces são quase a norma.

**Exemplo:** `IComparable`, `IDisposable`, `IEnumerable`. São contratos genéricos que classes muito diferentes podem implementar.

### Use **Classe Abstrata** quando:

- As classes filhas **compartilham código** que faz sentido ficar na base.
- Há uma relação clara de **"é um/uma"** entre as classes.
- Você quer **forçar** uma estrutura comum (atributos, construtor) em todas as filhas.
- Existe um **fluxo padrão** com pontos de personalização (Template Method).

**Exemplo:** `Stream` no .NET. Todas as classes que leem/escrevem dados (`FileStream`, `MemoryStream`) compartilham métodos como `Read`, `Write`, `Close`.

### Regra prática rápida

> 🎯 **Em dúvida? Comece com interface.**
>
> Use classe abstrata só quando perceber que está **repetindo o mesmo código** em várias filhas e quer extrair para a base.

---

## 8.7 Combinando os dois

Muitos projetos profissionais usam **ambos juntos**:

```csharp
public interface IFormaGeometrica
{
    double CalcularArea();
    string Nome { get; }
}

public abstract class FormaBase : IFormaGeometrica
{
    public abstract double CalcularArea();
    public abstract string Nome { get; }

    public void Imprimir()
    {
        Console.WriteLine($"{Nome}: área = {CalcularArea():F2}");
    }
}

public class Quadrado : FormaBase
{
    public double Lado { get; set; }
    public override double CalcularArea() => Lado * Lado;
    public override string Nome => "Quadrado";
}
```

A **interface** define o contrato; a **classe abstrata** dá uma implementação parcial reutilizável.

---

## 8.8 Dica de produtividade: `Ctrl + .`

> **Ponto de ouro do user (memorize):**
> O atalho **`Ctrl + .`** (Control + ponto) abre o menu de **ações rápidas, sugestões e ajustes de código** no VS Code e no Visual Studio.

Coloque o cursor em cima do nome da interface após o `:` da classe e aperte `Ctrl + .`. O VS Code oferece:

- **"Implement interface"** — gera todos os métodos automaticamente.
- **"Implement interface explicitly"** — versão mais formal.

```csharp
public class Retangulo : IFormaGeometrica  // cursor aqui, Ctrl + .
{
    // VS Code gera os métodos automaticamente
}
```

Funciona também para:

- Importar `using` que está faltando.
- Renomear variáveis.
- Extrair método.
- Gerar construtor a partir dos atributos.
- Converter `if/else` em `switch`.
- Muito mais.

> **Decore esse atalho.** Ele economiza horas.

---

## 8.9 Exemplo aplicado a Backend

Em uma API real, é assim que organizamos o código:

```csharp
// Contrato
public interface IClienteService
{
    Cliente BuscarPorId(int id);
    void Cadastrar(Cliente cliente);
}

// Implementação real (banco de dados)
public class ClienteService : IClienteService
{
    public Cliente BuscarPorId(int id)
    {
        // consulta o banco
        return new Cliente();
    }

    public void Cadastrar(Cliente cliente)
    {
        // grava no banco
    }
}

// O Controller depende da INTERFACE, não da classe concreta:
public class ClienteController
{
    private readonly IClienteService _service;

    public ClienteController(IClienteService service)
    {
        _service = service;
    }
}
```

Por que isso é genial? Veremos no **Capítulo 11** quando falarmos de **injeção de dependência**.

---

## 8.10 Resumo do capítulo

- **Interface** = contrato puro. Diz **o que**, não diz **como**.
- **Classe abstrata** = base parcial. Diz **o que** e às vezes **como**, mas não pode ser instanciada.
- Interfaces aceitam **implementação múltipla**; classes abstratas, **uma só** (herança).
- **Em dúvida, prefira interface.**
- **`Ctrl + .`** gera implementações automaticamente. Use sempre.

---

## 8.11 Exercícios

1. Crie a interface `IPagamento` com método `Processar(double valor)`. Implemente em `PagamentoCartao`, `PagamentoPix` e `PagamentoBoleto`.
2. Crie uma classe abstrata `Veiculo` com método abstrato `Acelerar()` e método concreto `Frear()`. Crie `Carro` e `Moto` herdeiros.
3. Crie uma interface `IComparavel<T>` com método `Comparar(T outro)` e implemente em uma classe `Produto` (compara por preço).
4. Faça uma classe abstrata `Funcionario` com `CalcularSalario()` abstrato e `RegistrarPonto()` concreto. Crie subclasses `CLT`, `PJ`, `Estagiario` com cálculos diferentes.
5. Combine interface + classe abstrata: `IAnimal`, `AnimalBase` (abstrata), `Cachorro`, `Gato`.

---

➡️ **Próximo capítulo:** [Capítulo 9 — Fundamentos de API e HTTP](09-Fundamentos-de-API-e-HTTP.md)
