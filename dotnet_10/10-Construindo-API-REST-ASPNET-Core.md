# Capítulo 10 — Construindo uma API REST com ASP.NET Core

> Hora de pôr a mão na massa! Vamos criar uma API completa, do zero, e entender cada peça.

---

## 10.1 O que é ASP.NET Core?

**ASP.NET Core** é o framework do .NET para construir **aplicações web e APIs**. Ele:

- É **multiplataforma** (Windows, Linux, Mac).
- É **muito rápido** (entre os mais performáticos do mundo).
- Já vem com tudo: roteamento, JSON, autenticação, injeção de dependência e suporte nativo a OpenAPI.
- Quando queremos uma tela interativa para testar a API, adicionamos o Swagger UI por pacote.

---

## 10.2 Criando o primeiro projeto

No terminal, em uma pasta nova:

```bash
dotnet new webapi -n MinhaApi --use-controllers --framework net10.0
cd MinhaApi
dotnet add package Microsoft.AspNetCore.OpenApi --version 10.0.8
dotnet add package Swashbuckle.AspNetCore.SwaggerUI --version 10.2.1
dotnet run
```

Você verá algo como:

```
Now listening on: http://localhost:5000
Now listening on: https://localhost:5001
```

No .NET 10, o template gera o documento OpenAPI em JSON. Com a configuração do próximo tópico, você terá:

- Documento OpenAPI: <https://localhost:5001/openapi/v1.json>
- Interface Swagger UI: <https://localhost:5001/swagger>

🎉 Você acabou de criar e rodar uma API com **interface de testes** automática.

---

## 10.3 Estrutura inicial do projeto

```
MinhaApi/
├── Controllers/
│   └── WeatherForecastController.cs
├── Properties/
│   └── launchSettings.json
├── appsettings.json
├── appsettings.Development.json
├── MinhaApi.csproj
├── Program.cs
└── WeatherForecast.cs
```

| Item | Função |
|---|---|
| **`Program.cs`** | Ponto de entrada e configuração da API. |
| **`Controllers/`** | Onde ficam os controladores (endpoints). |
| **`appsettings.json`** | Configurações (string de conexão, chaves, etc). |
| **`launchSettings.json`** | Configurações de execução local. |
| **`.csproj`** | Manifesto do projeto. |

---

## 10.4 Entendendo o `Program.cs`

Abra `Program.cs`. Em projetos modernos, ele é enxuto: primeiro registramos os serviços; depois ligamos o pipeline HTTP.

No .NET 10, use `AddOpenApi()` para gerar o documento OpenAPI e `MapOpenApi()` para expor o JSON. Para ter a página interativa `/swagger`, instale `Swashbuckle.AspNetCore.SwaggerUI` e aponte a UI para o documento gerado em `/openapi/v1.json`.

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1) Adicionar serviços ao container
builder.Services.AddControllers();
builder.Services.AddOpenApi();

var app = builder.Build();

// 2) Configurar o pipeline HTTP
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/openapi/v1.json", "MinhaApi v1");
    });
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

Em duas frases:

1. **Builder**: registramos quais "ferramentas" (serviços) a aplicação vai usar.
2. **App**: configuramos como cada requisição vai ser tratada (o **pipeline**).

Repare na separação:

- **OpenAPI** é a descrição formal da API em JSON.
- **Swagger UI** é a página bonita e interativa que lê esse JSON e permite executar requisições pelo navegador.
- Em .NET 10, essa separação fica mais explícita: o ASP.NET Core gera o documento; a UI é adicionada quando você precisa dela.

---

## 10.5 Criando o primeiro Controller

Vamos apagar `WeatherForecastController.cs` e `WeatherForecast.cs` e criar nosso próprio recurso: **Produtos**.

### Passo 1 — Criar a model

Crie a pasta `Models/` e um arquivo `Produto.cs`:

```csharp
namespace MinhaApi.Models;

public class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; } = string.Empty;
    public double Preco { get; set; }
}
```

### Passo 2 — Criar o Controller

Em `Controllers/`, crie `ProdutosController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using MinhaApi.Models;

namespace MinhaApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    // "Banco de dados" temporário em memória
    private static List<Produto> _produtos = new()
    {
        new Produto { Id = 1, Nome = "Notebook", Preco = 4500 },
        new Produto { Id = 2, Nome = "Mouse", Preco = 80 }
    };

    // GET /api/produtos
    [HttpGet]
    public IEnumerable<Produto> Listar()
    {
        return _produtos;
    }

    // GET /api/produtos/1
    [HttpGet("{id}")]
    public ActionResult<Produto> Buscar(int id)
    {
        var produto = _produtos.FirstOrDefault(p => p.Id == id);
        if (produto == null)
            return NotFound();
        return produto;
    }

    // POST /api/produtos
    [HttpPost]
    public ActionResult<Produto> Criar(Produto produto)
    {
        produto.Id = _produtos.Max(p => p.Id) + 1;
        _produtos.Add(produto);
        return CreatedAtAction(nameof(Buscar), new { id = produto.Id }, produto);
    }

    // PUT /api/produtos/1
    [HttpPut("{id}")]
    public IActionResult Atualizar(int id, Produto entrada)
    {
        var produto = _produtos.FirstOrDefault(p => p.Id == id);
        if (produto == null)
            return NotFound();

        produto.Nome = entrada.Nome;
        produto.Preco = entrada.Preco;
        return NoContent();
    }

    // DELETE /api/produtos/1
    [HttpDelete("{id}")]
    public IActionResult Remover(int id)
    {
        var produto = _produtos.FirstOrDefault(p => p.Id == id);
        if (produto == null)
            return NotFound();

        _produtos.Remove(produto);
        return NoContent();
    }
}
```

### Passo 3 — Rodar e testar

```bash
dotnet run
```

Acesse o Swagger UI: <https://localhost:5001/swagger>. Você vai ver os 5 endpoints prontos para testar pela interface.

Se quiser conferir o contrato bruto da API, acesse também <https://localhost:5001/openapi/v1.json>. Esse é o documento OpenAPI que a interface usa por baixo.

---

## 10.6 Anatomia do Controller

Vamos entender cada peça.

### Atributos da classe

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
```

| Atributo | Função |
|---|---|
| `[ApiController]` | Marca como controller de API. Habilita validação automática, mensagens de erro padronizadas, etc. |
| `[Route("api/[controller]")]` | Define o **prefixo** das rotas. `[controller]` é substituído pelo nome da classe sem "Controller" → `produtos`. |
| Herda de `ControllerBase` | Classe base que dá métodos como `Ok()`, `NotFound()`, `BadRequest()`. |

### Atributos dos métodos

```csharp
[HttpGet]            // GET /api/produtos
[HttpGet("{id}")]    // GET /api/produtos/{id}
[HttpPost]           // POST /api/produtos
[HttpPut("{id}")]    // PUT /api/produtos/{id}
[HttpDelete("{id}")] // DELETE /api/produtos/{id}
```

`{id}` é um **parâmetro de rota** que vira parâmetro do método.

### Tipos de retorno

| Retorno | Quando usar |
|---|---|
| `IEnumerable<T>` | Lista simples (status sempre 200). |
| `ActionResult<T>` | Quando você precisa retornar **dados ou um status code** específico. |
| `IActionResult` | Quando você só quer retornar um status code (sem dados). |

### Métodos auxiliares de status

```csharp
return Ok(produto);                // 200
return Created(url, produto);      // 201
return NoContent();                // 204
return BadRequest("mensagem");     // 400
return Unauthorized();             // 401
return Forbid();                   // 403
return NotFound();                 // 404
return Conflict("mensagem");       // 409
```

---

## 10.7 Recebendo dados de diferentes lugares

ASP.NET Core faz **bind automático** dos parâmetros, mas você pode ser explícito:

```csharp
[HttpGet("{id}")]   // Parâmetro de ROTA
public IActionResult Buscar([FromRoute] int id) { }

[HttpGet]   // Query string: /api/produtos?categoria=...
public IActionResult Listar(
    [FromQuery] string? categoria,
    [FromQuery] double? precoMax)
{ }

[HttpPost]   // BODY (JSON)
public IActionResult Criar([FromBody] Produto produto) { }

// Header
public IActionResult Algo([FromHeader(Name = "X-Tenant")] string tenant) { }
```

Em geral, com `[ApiController]`, o framework **deduz**:

- Tipos primitivos (int, string) na rota → `[FromRoute]`.
- Tipos primitivos sem rota → `[FromQuery]`.
- Tipos complexos → `[FromBody]`.

---

## 10.8 Validação automática

Com **Data Annotations**, você pode validar models facilmente:

```csharp
using System.ComponentModel.DataAnnotations;

public class Produto
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Nome é obrigatório")]
    [StringLength(100, MinimumLength = 2)]
    public string Nome { get; set; } = string.Empty;

    [Range(0.01, 1_000_000, ErrorMessage = "Preço inválido")]
    public double Preco { get; set; }
}
```

Se vier inválido, a API responde automaticamente com **400 Bad Request** e um JSON detalhando os erros. **Sem você escrever uma linha de código de validação.**

### Validações comuns

| Atributo | Função |
|---|---|
| `[Required]` | Campo obrigatório |
| `[StringLength(max, MinimumLength = min)]` | Tamanho de texto |
| `[Range(min, max)]` | Faixa numérica |
| `[EmailAddress]` | Formato de e-mail |
| `[RegularExpression(...)]` | Regex |
| `[MinLength]` / `[MaxLength]` | Tamanho de coleções |

---

## 10.9 Testando com Swagger UI

O Swagger UI é uma página interativa que **lê o documento OpenAPI** gerado a partir dos seus controllers e monta uma UI de testes automática.

Ao rodar `dotnet run`, abra `https://localhost:PORTA/swagger`:

1. Clique em um endpoint → "Try it out".
2. Preencha os parâmetros.
3. "Execute" → veja a resposta.

Excelente para desenvolver e documentar sem precisar de Postman.

---

## 10.10 Configurações: `appsettings.json`

Aqui ficam coisas que **mudam por ambiente** (string de conexão, chaves de API, URLs externas):

```json
{
  "Logging": {
    "LogLevel": { "Default": "Information" }
  },
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MinhaApp;Trusted_Connection=true"
  },
  "ApiExterna": {
    "Url": "https://api.terceiros.com",
    "ApiKey": "abc123"
  }
}
```

Lendo do código:

```csharp
public class MeuController : ControllerBase
{
    private readonly IConfiguration _config;

    public MeuController(IConfiguration config)
    {
        _config = config;
    }

    [HttpGet]
    public IActionResult Get()
    {
        string url = _config["ApiExterna:Url"];
        return Ok(url);
    }
}
```

> **Nunca** coloque senhas reais no `appsettings.json` versionado. Use **User Secrets** ou variáveis de ambiente em produção.

---

## 10.11 CORS — permitir chamadas do frontend

Por padrão, navegadores bloqueiam chamadas entre domínios diferentes. Para permitir que seu frontend (rodando em outro endereço) chame sua API:

```csharp
// em Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("Frontend", policy =>
        policy.WithOrigins("http://localhost:3000")
              .AllowAnyHeader()
              .AllowAnyMethod());
});

// ...

app.UseCors("Frontend");
```

---

## 10.12 Exemplo: Adicionando logs

ASP.NET Core já vem com sistema de log. Receba via injeção:

```csharp
public class ProdutosController : ControllerBase
{
    private readonly ILogger<ProdutosController> _logger;

    public ProdutosController(ILogger<ProdutosController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IEnumerable<Produto> Listar()
    {
        _logger.LogInformation("Listando produtos...");
        return _produtos;
    }
}
```

Verá os logs no terminal.

---

## 10.13 O ciclo de vida de uma requisição

Quando alguém faz `GET /api/produtos/1`:

1. ASP.NET Core recebe a requisição HTTP.
2. Passa pelo **pipeline de middlewares** (HTTPS, autenticação, autorização…).
3. Identifica o **controller** e **action** pela rota.
4. Faz o **binding** dos parâmetros.
5. Roda a **validação** (se aplicável).
6. Executa o método do controller.
7. Serializa o retorno como **JSON**.
8. Devolve a resposta HTTP.

Isso tudo acontece em **milissegundos**.

---

## 10.14 Resumo do capítulo

- Criamos uma API com `dotnet new webapi`.
- Vimos a estrutura: `Program.cs`, controllers, models, configs.
- Construímos um CRUD completo com 5 endpoints.
- Aprendemos atributos: `[ApiController]`, `[Route]`, `[HttpGet]`, etc.
- Conhecemos `IActionResult`, `ActionResult<T>` e métodos como `Ok`, `NotFound`, `BadRequest`.
- Vimos validação automática com **Data Annotations**.
- Testamos tudo no **Swagger UI**.

---

## 10.15 Exercícios

1. Crie um controller `ClientesController` com CRUD completo (model `Cliente` com `Id`, `Nome`, `Email`).
2. Adicione validação no `Cliente`: nome obrigatório, e-mail obrigatório e válido.
3. Adicione um endpoint extra: `GET /api/clientes/buscar?nome=...` (filtra por nome).
4. Adicione um endpoint para retornar a quantidade total: `GET /api/clientes/contagem`.
5. Teste tudo no Swagger UI.

---

➡️ **Próximo capítulo:** [Capítulo 11 — Arquitetura: Controllers, Services, DTOs e Models](11-Arquitetura-Controllers-Services-DTOs-Models.md)
