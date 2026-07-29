# Capítulo 14 — Autenticação com JWT

> "Quem é você? Você pode entrar?" Toda API séria precisa responder essas duas perguntas. JWT é o padrão **mais usado** hoje.

---

## 14.1 Autenticação vs. Autorização

São **duas coisas diferentes**, mesmo que pareçam parecidas:

| Conceito | Pergunta que responde |
|---|---|
| **Autenticação** | **Quem é você?** (verificar a identidade) |
| **Autorização** | **O que você pode fazer?** (verificar permissões) |

Você pode estar autenticado (sabemos que é a Maria) mas não autorizado (Maria não pode deletar usuários).

---

## 14.2 O problema sem autenticação

Sem autenticação, qualquer pessoa que conhecer a URL da sua API pode:

- Listar todos os usuários cadastrados.
- Apagar dados.
- Consumir recursos pagos (envio de e-mails, gateways, etc.).

**Solução**: exigir que o cliente "**prove quem é**" antes de cada operação sensível.

---

## 14.3 O que é JWT?

**JWT** = **JSON Web Token**. É um **token compacto, autocontido e assinado** que carrega informações sobre o usuário.

### Anatomia de um JWT

Um JWT tem **3 partes**, separadas por ponto:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjMiLCJuYW1lIjoiSm9obiJ9.SflKxwRJSMeKKF2QT4f...
└──── HEADER ────┘.└────── PAYLOAD ──────┘.└────────── SIGNATURE ──────────┘
```

### 1. Header
Diz o tipo de token e o algoritmo de assinatura:
```json
{ "alg": "HS256", "typ": "JWT" }
```

### 2. Payload
Contém os **claims** — informações sobre o usuário:
```json
{
  "sub": "123",
  "name": "Maria",
  "exp": 1735689600
}
```

### 3. Signature
Assinatura criptográfica que **garante a integridade** do token:
```
HMAC_SHA256(base64(header) + "." + base64(payload), CHAVE_SECRETA)
```

> **Em uma frase:** o JWT é um "crachá assinado" — qualquer servidor com a chave secreta pode **verificar** se o crachá é genuíno.

### Como JWT funciona na prática

1. Usuário faz **login** com usuário/senha.
2. Servidor valida e gera um **JWT** assinado, devolvendo ao cliente.
3. Cliente armazena o token (geralmente em `localStorage` ou cookie).
4. A cada requisição protegida, cliente envia o token no **header**:
   ```
   Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
   ```
5. Servidor **valida a assinatura** e libera (ou bloqueia) a requisição.

### Por que JWT é "stateless"?

Porque o servidor **não precisa guardar sessão**. A informação está **dentro do próprio token**. Isso facilita escalar (vários servidores não precisam compartilhar memória).

---

## 14.4 Pacotes NuGet

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer --version 10.0.8
dotnet add package Microsoft.AspNetCore.OpenApi --version 10.0.8
dotnet add package Microsoft.IdentityModel.Tokens --version 8.19.1
dotnet add package Swashbuckle.AspNetCore.SwaggerUI --version 10.2.1
dotnet add package System.IdentityModel.Tokens.Jwt --version 8.19.1
```

Os três primeiros pacotes cuidam da autenticação e da geração/validação dos tokens. `Microsoft.AspNetCore.OpenApi` gera o documento OpenAPI no padrão atual do ASP.NET Core 10. `Swashbuckle.AspNetCore.SwaggerUI` adiciona a tela interativa `/swagger`, que não é a mesma coisa que o documento OpenAPI: ela apenas consome esse documento para deixar os testes mais confortáveis.

---

## 14.5 Configurando o JWT no `appsettings.json`

```json
{
  "JwtSettings": {
    "Key": "sua-chave-secreta-muito-longa-aqui-com-no-minimo-32-caracteres",
    "Issuer": "Movies.API",
    "Audience": "Movies.API",
    "DurationMinutes": 60
  }
}
```

| Campo | Significado |
|---|---|
| `Key` | **Chave secreta** usada para assinar o token. Mantenha em **segredo absoluto**. |
| `Issuer` | Quem emite o token (sua API). |
| `Audience` | Quem deve receber o token (geralmente sua API mesmo). |
| `DurationMinutes` | Tempo de validade. **Curto é melhor** (1h é razoável). |

> ⚠️ A `Key` precisa ter **pelo menos 32 caracteres** (256 bits) para o algoritmo HS256.

### Classe `JwtSettings` correspondente

```csharp
public class JwtSettings
{
    public string Key { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public double DurationMinutes { get; set; }
}
```

---

## 14.6 Gerador de tokens — `JwtAuthManager`

Classe responsável por **criar o token** quando o login é bem-sucedido:

```csharp
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

public static class JwtAuthManager
{
    public static string GenerateToken(string userName)
    {
        // 1) Lê configuração
        var configuration = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json")
            .Build();

        var jwtSettings = configuration.GetSection("JwtSettings").Get<JwtSettings>()!;

        // 2) Prepara a chave
        var key = Encoding.ASCII.GetBytes(jwtSettings.Key);

        // 3) Define claims (informações dentro do token)
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, userName)
        };

        // 4) Configura o token
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddMinutes(jwtSettings.DurationMinutes),
            Issuer = jwtSettings.Issuer,
            Audience = jwtSettings.Audience,
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(key),
                SecurityAlgorithms.HmacSha256)
        };

        // 5) Gera o token
        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }
}
```

### O que são "claims"?

**Claims** são **afirmações** sobre o usuário, dentro do token. Você pode incluir qualquer coisa:

```csharp
new Claim(ClaimTypes.Name, "maria"),
new Claim(ClaimTypes.Email, "maria@email.com"),
new Claim(ClaimTypes.Role, "admin"),
new Claim("departamento", "TI")
```

Depois, dentro de um Controller protegido, você consegue ler o usuário com `User.Identity?.Name`.

---

## 14.7 Configurando o JWT no `Program.cs`

Aqui ensinamos a aplicação a **validar tokens recebidos**:

```csharp
using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// 1) Lê configurações do JWT
var jwtSettings = builder.Configuration.GetSection("JwtSettings").Get<JwtSettings>()
    ?? throw new InvalidOperationException("JwtSettings não configurado.");

// 2) Registra autenticação JWT
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtSettings.Issuer,
            ValidAudience = jwtSettings.Audience,
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(jwtSettings.Key))
        };
    });

var app = builder.Build();

// 3) ORDEM OBRIGATÓRIA
app.UseAuthentication();   // primeiro: identifica
app.UseAuthorization();    // depois: autoriza

app.MapControllers();
app.Run();
```

> ⚠️ **A ordem importa**: `UseAuthentication` **antes** de `UseAuthorization`. Senão, a autorização nunca encontra o usuário.

### O que cada validação faz?

| Validação | Verifica |
|---|---|
| `ValidateIssuer` | O token foi emitido por **quem deveria** (`Issuer` correto). |
| `ValidateAudience` | O token é destinado **a esta API** (`Audience` correto). |
| `ValidateLifetime` | O token **não expirou**. |
| `ValidateIssuerSigningKey` | A **assinatura** é válida (não foi adulterado). |

---

## 14.8 Protegendo endpoints com `[Authorize]`

Após configurado, basta adicionar o atributo:

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]   // ← TODOS os endpoints exigem JWT válido
public class UserController : ControllerBase
{
    [HttpGet]
    public IActionResult Listar() { ... }
}
```

Ou em endpoints específicos:

```csharp
public class ProdutosController : ControllerBase
{
    [HttpGet]   // público
    public IActionResult Listar() { ... }

    [HttpPost]
    [Authorize]   // só este exige login
    public IActionResult Criar() { ... }
}
```

### Permitir acesso público em controller protegido

```csharp
[Authorize]
public class UserController : ControllerBase
{
    [HttpPost]
    [AllowAnonymous]   // ← exceção: este é público
    public IActionResult Cadastrar() { ... }
}
```

---

## 14.9 Endpoint de login

```csharp
[ApiController]
[Route("api/[controller]")]
public class LoginController : ControllerBase
{
    [HttpPost]
    public IActionResult Login([FromBody] AuthRequest request)
    {
        using var connection = new DataContext();

        var hashEntrada = PasswordEncryptor.EncryptPassword(request.Password);

        var user = connection.Users
            .AsNoTracking()
            .FirstOrDefault(u => u.Username == request.Username
                              && u.Password == hashEntrada);

        if (user == null)
            return BadRequest("Login failed!");

        var token = JwtAuthManager.GenerateToken(user.Username);
        return Ok($"Login successful! Jwt: {token}");
    }
}
```

Fluxo:
1. Recebe usuário + senha.
2. Aplica o **mesmo hash** que foi usado ao salvar.
3. Compara com o que está no banco.
4. Se bater, gera o JWT.

---

## 14.10 Criptografando senhas

> ⚠️ **NUNCA armazene senhas em texto puro no banco**. Sempre **hasheie**.

Versão didática (SHA-256 + Base64):

```csharp
using System.Security.Cryptography;
using System.Text;

public static class PasswordEncryptor
{
    public static string EncryptPassword(string password)
    {
        if (string.IsNullOrEmpty(password))
            throw new ArgumentException("Senha não pode ser vazia");

        using var sha256 = SHA256.Create();
        var bytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(password));
        return Convert.ToBase64String(bytes);
    }

    public static bool VerifyPassword(string password, string hashed)
    {
        return EncryptPassword(password) == hashed;
    }
}
```

> 🔒 **Em produção**, use **BCrypt**, **Argon2** ou **PBKDF2** com **salt** — são resistentes a ataques de força bruta. Pacote recomendado: `BCrypt.Net-Next`. SHA-256 puro é didático, **não profissional**.

---

## 14.11 Configurando OpenAPI e Swagger UI para JWT

No ASP.NET Core 10, o caminho recomendado é separar duas coisas:

- **OpenAPI**: o documento JSON que descreve seus endpoints.
- **Swagger UI**: a página interativa que lê esse documento e permite testar a API.

Para que a UI exiba o botão **Authorize** e envie o header `Authorization: Bearer <token>`, registre um transformador de documento OpenAPI. O `AddOpenApi(...)` entra antes de `var app = builder.Build();`; já `MapOpenApi()` e `UseSwaggerUI(...)` entram depois que o `app` foi criado.

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.OpenApi;
using Microsoft.OpenApi;

builder.Services.AddOpenApi(options =>
{
    options.AddDocumentTransformer<BearerSecuritySchemeTransformer>();
});

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/openapi/v1.json", "API v1");
    });
}

internal sealed class BearerSecuritySchemeTransformer(
    IAuthenticationSchemeProvider authenticationSchemeProvider) : IOpenApiDocumentTransformer
{
    public async Task TransformAsync(
        OpenApiDocument document,
        OpenApiDocumentTransformerContext context,
        CancellationToken cancellationToken)
    {
        var authenticationSchemes = await authenticationSchemeProvider.GetAllSchemesAsync();

        if (!authenticationSchemes.Any(authScheme => authScheme.Name == JwtBearerDefaults.AuthenticationScheme))
            return;

        document.Components ??= new OpenApiComponents();
        document.Components.SecuritySchemes = new Dictionary<string, IOpenApiSecurityScheme>
        {
            [JwtBearerDefaults.AuthenticationScheme] = new OpenApiSecurityScheme
            {
                Type = SecuritySchemeType.Http,
                Scheme = "bearer",
                BearerFormat = "JWT",
                In = ParameterLocation.Header,
                Description = "Cole apenas o token JWT. A UI enviará o header Authorization: Bearer <token>."
            }
        };

        foreach (var pathItem in document.Paths.Values)
        {
            if (pathItem.Operations is null)
                continue;

            foreach (var operation in pathItem.Operations.Values)
            {
                operation.Security ??= [];
                operation.Security.Add(new OpenApiSecurityRequirement
                {
                    [new OpenApiSecuritySchemeReference(JwtBearerDefaults.AuthenticationScheme, document)] = []
                });
            }
        }
    }
}
```

Agora, no Swagger UI, aparece um botão **🔒 Authorize** no topo. Cole **somente o token JWT**, sem escrever `Bearer` antes. Como o esquema foi configurado como `type: http` e `scheme: bearer`, a própria UI monta o header correto:

```http
Authorization: Bearer <seu_token>
```

---

## 14.12 Fluxo completo (visual)

```
┌──────────┐    POST /api/Login                    ┌──────────┐
│ Cliente  │  ─────────────────────────────────►   │   API    │
│          │  { username, password }                │          │
│          │                                        │  valida  │
│          │  ◄────────────────  JWT  ─────────     │  gera    │
└────┬─────┘                                        └──────────┘
     │ guarda token
     ▼
┌──────────┐    GET /api/User                      ┌──────────┐
│ Cliente  │  ─────────────────────────────────►   │   API    │
│          │  Authorization: Bearer eyJ...         │ valida   │
│          │                                        │ token    │
│          │  ◄────────  200 OK + dados   ─────     │          │
└──────────┘                                        └──────────┘
```

---

## 14.13 Boas práticas com JWT

1. **HTTPS sempre.** Token interceptado = conta invadida.
2. **Tempo de vida curto** (15min a 1h). Tokens longos são perigosos.
3. **Refresh tokens** para sessão longa (assunto avançado).
4. **Não coloque dados sensíveis** no payload — ele é apenas codificado em Base64, não criptografado. Qualquer um pode ler!
5. **Nunca exponha a `Key`** em commit ou logs.
6. **Use chaves diferentes** em dev e prod.
7. **Revogação**: JWT é stateless — para invalidar antes de expirar, precisa de uma "blacklist" no servidor.

---

## 14.14 Resumo do capítulo

- **Autenticação** = quem você é. **Autorização** = o que pode fazer.
- **JWT** é um token assinado contendo dados do usuário (claims).
- Login → gera token. Requisições seguintes → enviam token no header.
- `[Authorize]` protege endpoints. `[AllowAnonymous]` libera dentro de protegido.
- **HTTPS** é mandatório em produção.
- Senhas: SEMPRE com hash. Em produção, **BCrypt/Argon2**, não SHA-256 puro.

---

## 14.15 Exercícios

1. Crie a classe `JwtSettings` e configure no `appsettings.json`.
2. Implemente `PasswordEncryptor` e teste no console.
3. Implemente `JwtAuthManager.GenerateToken` e gere um token. Cole em <https://jwt.io> e veja o conteúdo decodificado.
4. Configure o JWT no `Program.cs` da sua API (Cap. 11).
5. Proteja o `ClientesController` com `[Authorize]` e crie um `LoginController` que devolve token.
6. Teste tudo no Swagger UI: tente acessar sem token → deve dar 401. Faça login, copie o token, autorize colando apenas o JWT, e teste novamente.

---

➡️ **Próximo capítulo:** [Capítulo 15 — Construindo a Movies.API passo a passo](15-Construindo-Movies-API-Passo-a-Passo.md)
