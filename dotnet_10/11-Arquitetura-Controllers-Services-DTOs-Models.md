# Capítulo 11 — Arquitetura: Controllers, Services, DTOs e Models

> Colocar tudo no Controller é fácil — e errado. Aqui você aprende a **organizar** seu projeto como os profissionais.

---

## 11.1 O problema de "tudo no Controller"

No capítulo anterior, todo o código estava dentro do Controller. Funciona, mas:

- O Controller fica **enorme**.
- Mistura **regra de negócio**, **acesso a dados** e **detalhes HTTP**.
- Fica **difícil de testar**.
- **Não dá para reutilizar** nada.
- Mudar o banco de dados exige reescrever o Controller.

**Solução:** separar em **camadas**, cada uma com **uma responsabilidade**.

---

## 11.2 As camadas típicas

```
┌─────────────────────────────────────────┐
│     Controller (recebe HTTP)            │  ← entende API/HTTP
├─────────────────────────────────────────┤
│     Service (regras de negócio)         │  ← lógica do sistema
├─────────────────────────────────────────┤
│     Repository (acesso a dados)         │  ← fala com banco
├─────────────────────────────────────────┤
│     Banco de dados                      │
└─────────────────────────────────────────┘
```

E circulam **objetos**:

- **Model / Entity**: representa o dado real.
- **DTO**: representa o que entra/sai pela API.

---

## 11.3 Models (Entidades)

São as classes que representam **as coisas do sistema**: `Produto`, `Cliente`, `Pedido`. Frequentemente espelham tabelas do banco.

```csharp
// Models/Produto.cs
public class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; } = string.Empty;
    public double Preco { get; set; }
    public int EstoqueAtual { get; set; }
    public DateTime CriadoEm { get; set; }
    public string ChaveInternaSeguranca { get; set; } = string.Empty;
}
```

> Models podem conter **dados sensíveis ou internos** que não devem aparecer na API. Por isso existem os DTOs.

---

## 11.4 DTOs (Data Transfer Objects)

Um **DTO** é uma classe que existe **só para transitar dados pela API**.

### Por que separar Model e DTO?

| Motivo | Explicação |
|---|---|
| **Segurança** | Você só expõe o que quer expor. |
| **Estabilidade** | Você muda o Model sem quebrar quem consome a API. |
| **Validação específica** | Os campos requeridos para criar são diferentes dos para atualizar. |
| **Performance** | Trafega só o necessário. |

### Exemplo

```csharp
// DTOs/CriarProdutoDto.cs
public class CriarProdutoDto
{
    [Required]
    public string Nome { get; set; } = string.Empty;

    [Range(0.01, 1_000_000)]
    public double Preco { get; set; }
}

// DTOs/ProdutoRespostaDto.cs
public class ProdutoRespostaDto
{
    public int Id { get; set; }
    public string Nome { get; set; } = string.Empty;
    public double Preco { get; set; }
    // ChaveInternaSeguranca NÃO aparece aqui!
}
```

### Convertendo Model ↔ DTO

Manualmente:

```csharp
ProdutoRespostaDto MapearParaDto(Produto p) => new()
{
    Id = p.Id,
    Nome = p.Nome,
    Preco = p.Preco
};
```

Em projetos maiores, usa-se bibliotecas como **AutoMapper** ou **Mapster**.

---

## 11.5 Services (Camada de negócio)

O **Service** contém a **lógica do sistema**: regras de negócio, validações cruzadas, orquestração.

### Interface primeiro

```csharp
// Services/IProdutoService.cs
public interface IProdutoService
{
    IEnumerable<ProdutoRespostaDto> Listar();
    ProdutoRespostaDto? Buscar(int id);
    ProdutoRespostaDto Criar(CriarProdutoDto dto);
    bool Atualizar(int id, CriarProdutoDto dto);
    bool Remover(int id);
}
```

### Implementação

```csharp
// Services/ProdutoService.cs
public class ProdutoService : IProdutoService
{
    private readonly IProdutoRepository _repo;

    public ProdutoService(IProdutoRepository repo)
    {
        _repo = repo;
    }

    public IEnumerable<ProdutoRespostaDto> Listar()
    {
        return _repo.ListarTodos().Select(MapearParaDto);
    }

    public ProdutoRespostaDto? Buscar(int id)
    {
        var produto = _repo.BuscarPorId(id);
        return produto == null ? null : MapearParaDto(produto);
    }

    public ProdutoRespostaDto Criar(CriarProdutoDto dto)
    {
        // Regra de negócio: nome único
        if (_repo.ExistePorNome(dto.Nome))
            throw new InvalidOperationException("Produto com esse nome já existe");

        var produto = new Produto
        {
            Nome = dto.Nome,
            Preco = dto.Preco,
            EstoqueAtual = 0,
            CriadoEm = DateTime.UtcNow
        };
        _repo.Adicionar(produto);
        return MapearParaDto(produto);
    }

    public bool Atualizar(int id, CriarProdutoDto dto)
    {
        var produto = _repo.BuscarPorId(id);
        if (produto == null) return false;

        produto.Nome = dto.Nome;
        produto.Preco = dto.Preco;
        _repo.Atualizar(produto);
        return true;
    }

    public bool Remover(int id)
    {
        var produto = _repo.BuscarPorId(id);
        if (produto == null) return false;
        _repo.Remover(produto);
        return true;
    }

    private static ProdutoRespostaDto MapearParaDto(Produto p) => new()
    {
        Id = p.Id,
        Nome = p.Nome,
        Preco = p.Preco
    };
}
```

---

## 11.6 Repositories (Acesso a dados)

O **Repository** isola o **acesso ao banco** (ou outra fonte). O resto do sistema **não sabe** se é SQL Server, Mongo ou um arquivo CSV.

### Interface

```csharp
public interface IProdutoRepository
{
    IEnumerable<Produto> ListarTodos();
    Produto? BuscarPorId(int id);
    bool ExistePorNome(string nome);
    void Adicionar(Produto produto);
    void Atualizar(Produto produto);
    void Remover(Produto produto);
}
```

### Implementação em memória (para começar)

```csharp
public class ProdutoRepositoryMemoria : IProdutoRepository
{
    private static readonly List<Produto> _produtos = new();
    private static int _proximoId = 1;

    public IEnumerable<Produto> ListarTodos() => _produtos.ToList();

    public Produto? BuscarPorId(int id) =>
        _produtos.FirstOrDefault(p => p.Id == id);

    public bool ExistePorNome(string nome) =>
        _produtos.Any(p => p.Nome.Equals(nome, StringComparison.OrdinalIgnoreCase));

    public void Adicionar(Produto produto)
    {
        produto.Id = _proximoId++;
        _produtos.Add(produto);
    }

    public void Atualizar(Produto produto)
    {
        // Como é a mesma instância em memória, não precisa fazer nada extra.
    }

    public void Remover(Produto produto) => _produtos.Remove(produto);
}
```

> No mundo real, aqui usaríamos **Entity Framework Core** ou **Dapper** para falar com um banco de dados de verdade. Os princípios são os mesmos.

---

## 11.7 Controller enxuto

Com Service e Repository prontos, o Controller fica **fininho**:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    private readonly IProdutoService _service;

    public ProdutosController(IProdutoService service)
    {
        _service = service;
    }

    [HttpGet]
    public ActionResult<IEnumerable<ProdutoRespostaDto>> Listar()
        => Ok(_service.Listar());

    [HttpGet("{id}")]
    public ActionResult<ProdutoRespostaDto> Buscar(int id)
    {
        var produto = _service.Buscar(id);
        return produto == null ? NotFound() : Ok(produto);
    }

    [HttpPost]
    public ActionResult<ProdutoRespostaDto> Criar(CriarProdutoDto dto)
    {
        try
        {
            var criado = _service.Criar(dto);
            return CreatedAtAction(nameof(Buscar), new { id = criado.Id }, criado);
        }
        catch (InvalidOperationException ex)
        {
            return Conflict(ex.Message);
        }
    }

    [HttpPut("{id}")]
    public IActionResult Atualizar(int id, CriarProdutoDto dto)
        => _service.Atualizar(id, dto) ? NoContent() : NotFound();

    [HttpDelete("{id}")]
    public IActionResult Remover(int id)
        => _service.Remover(id) ? NoContent() : NotFound();
}
```

O Controller só **traduz HTTP ↔ Service**. Toda lógica fica no Service.

---

## 11.8 Injeção de Dependência

Você reparou que o Controller recebe `IProdutoService` no construtor? E o Service recebe `IProdutoRepository`? Isso é **Injeção de Dependência (DI)**.

### O conceito

Em vez de a classe **criar** suas dependências (`new ProdutoService()`), ela **recebe** prontas. Vantagens:

- **Trocar implementações** facilmente (ex: trocar repositório de memória por banco real).
- **Testar** com facilidade (passar uma versão fake).
- **Acoplamento baixo** — o Controller depende de **`IProdutoService`** (contrato), não de uma classe específica.

### Configurando no `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddOpenApi();

// Registrando os serviços
builder.Services.AddScoped<IProdutoService, ProdutoService>();
builder.Services.AddSingleton<IProdutoRepository, ProdutoRepositoryMemoria>();

var app = builder.Build();
// ...
```

Se este projeto também expuser a tela de testes, mantenha o mesmo padrão do capítulo anterior: `app.MapOpenApi()` para o documento JSON e `app.UseSwaggerUI(...)` apontando para `/openapi/v1.json`.

### Tempos de vida

| Método | Quando uma nova instância é criada |
|---|---|
| `AddSingleton` | **Uma vez** durante toda a vida da aplicação |
| `AddScoped` | **Uma vez por requisição HTTP** |
| `AddTransient` | **Toda vez** que é solicitada |

> **Padrão prático:**
> - Controllers e Services → `Scoped`.
> - Repositórios baseados em DbContext → `Scoped`.
> - Caches/utilitários sem estado → `Singleton`.

> 📌 **Nota sobre o projeto Movies.API (Cap. 15)**
>
> Para fins didáticos, o projeto **Movies.API** que vocês irão construir adota uma alternativa **mais simples**: cada método do Service instancia o `DataContext` manualmente com `using var connection = new DataContext();`, sem registrar o `DbContext` na DI.
>
> Essa abordagem é **válida e funcional** para projetos pequenos/aprendizado, mas a forma **profissional e recomendada** em projetos reais é a que vimos aqui (registrar `AddDbContext` + injetar via construtor). Lembre-se disso quando chegar no Capítulo 15.

---

## 11.9 Estrutura de pastas recomendada

```
MinhaApi/
├── Controllers/
│   └── ProdutosController.cs
├── Services/
│   ├── IProdutoService.cs
│   └── ProdutoService.cs
├── Repositories/
│   ├── IProdutoRepository.cs
│   └── ProdutoRepositoryMemoria.cs
├── Models/
│   └── Produto.cs
├── DTOs/
│   ├── CriarProdutoDto.cs
│   └── ProdutoRespostaDto.cs
├── Program.cs
└── appsettings.json
```

Em projetos maiores, vários times usam **estrutura por feature**:

```
Features/
├── Produtos/
│   ├── ProdutosController.cs
│   ├── ProdutoService.cs
│   ├── ProdutoRepository.cs
│   ├── Produto.cs
│   └── Dtos/
└── Clientes/
    └── ...
```

Não há "uma forma certa". A regra é: **seja consistente**.

---

## 11.10 Tratamento de erros centralizado

Em vez de cada Controller capturar exceções, crie um **middleware**:

```csharp
public class ManipuladorDeExcecoes
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ManipuladorDeExcecoes> _logger;

    public ManipuladorDeExcecoes(RequestDelegate next, ILogger<ManipuladorDeExcecoes> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try { await _next(context); }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Erro não tratado");
            context.Response.StatusCode = 500;
            await context.Response.WriteAsJsonAsync(new
            {
                erro = "Ocorreu um erro inesperado.",
                detalhes = ex.Message
            });
        }
    }
}

// No Program.cs:
app.UseMiddleware<ManipuladorDeExcecoes>();
```

---

## 11.11 Exemplo completo de fluxo

Cliente envia: `POST /api/produtos` com `{ "nome": "Mouse", "preco": 80 }`

1. **Pipeline** ASP.NET → roteamento.
2. `ProdutosController.Criar(dto)` recebe e valida (`[ApiController]` cuida).
3. Controller chama `_service.Criar(dto)`.
4. Service verifica regra de negócio (nome único).
5. Service mapeia DTO → Model.
6. Service chama `_repo.Adicionar(produto)`.
7. Repository salva.
8. Service mapeia Model → ResponseDto.
9. Controller retorna `201 Created` com o DTO.

Cada camada com **um único papel**. **Esse é o segredo.**

---

## 11.12 Resumo do capítulo

- **Controllers** lidam só com HTTP.
- **Services** contêm a regra de negócio.
- **Repositories** acessam o banco.
- **Models** representam dados internos; **DTOs** representam dados que entram/saem.
- **Injeção de dependência** mantém tudo desacoplado.
- Estrutura de pastas clara facilita manutenção.

---

## 11.13 Exercícios

1. Refatore o `ClientesController` do capítulo anterior em **Controller + Service + Repository (em memória) + DTOs**.
2. Adicione regra de negócio no `ClienteService`: não permitir e-mails duplicados.
3. Crie dois DTOs: `CriarClienteDto` e `ClienteRespostaDto`. Garanta que campos sensíveis (se houver) não vão na resposta.
4. Configure DI no `Program.cs` com `Scoped` para Service e `Singleton` para Repository.
5. Crie um middleware de tratamento de exceções como mostrado.

---

➡️ **Próximo capítulo:** [Capítulo 12 — Boas Práticas e Clean Code](12-Boas-Praticas-e-Clean-Code.md)
