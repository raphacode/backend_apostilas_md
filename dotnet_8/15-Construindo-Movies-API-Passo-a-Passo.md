# Capítulo 15 — Construindo a Movies.API Passo a Passo

> Hora de juntar **TUDO**. Vamos construir, do zero ao deploy local, uma API completa de filmes com **PostgreSQL**, **Entity Framework Core**, **JWT** e **Swagger**, exatamente na ordem que profissionais constroem.
>
> **Este capítulo é o projeto final da apostila.** Siga em ordem, sem pular nenhum passo.

---

## 15.1 O que vamos construir

A **Movies.API** terá:

- **Domínio público**: CRUD de filmes (`Movie`).
- **Domínio protegido**: CRUD de usuários (`User`) — exige JWT.
- **Login**: gera o JWT após validar usuário/senha.
- **HealthCheck**: endpoint para verificar se a API está no ar.

### Stack

- **.NET 8** + **C# 12**
- **Entity Framework Core 9** + **PostgreSQL** (porta 5432)
- **JWT Bearer** para autenticação
- **Swagger** para documentação interativa

---

## 15.2 Princípio fundamental: ordem de criação

> ⚠️ **Crie cada arquivo SOMENTE depois que tudo que ele depende já existir.**
>
> Os Controllers e o `Program.cs` ficam **por último**. Sempre.

Vamos seguir essa **ordem de 25 passos**. Cada arquivo é um passo. Não pule.

---

## 15.3 Pré-requisitos

### 15.3.1 Comece por aqui: links oficiais

Este passo a passo considera um computador com **Windows de 64 bits (x64)** e um usuário com permissão de administrador. Faça os downloads abaixo antes de criar a `Movies.API`:

| Ordem | Programa | Link oficial | O que baixar |
|---|---|---|---|
| 1 | **Visual Studio Community** | <https://visualstudio.microsoft.com/pt-br/vs/community/> | A edição **Community** oferecida no botão de download |
| 2 | **SDK do .NET 8** | <https://dotnet.microsoft.com/pt-br/download/dotnet/8.0> | O instalador do **SDK 8.0** para Windows, e não apenas o Runtime |
| 3 | **PostgreSQL + pgAdmin 4** | <https://www.enterprisedb.com/downloads/postgres-postgresql-downloads> | O instalador do PostgreSQL para **Windows x86-64**; o pgAdmin será selecionado dentro dele |

> 🎯 **Configuração obrigatória deste material**
>
> - SDK usado pelo projeto: **.NET 8**
> - Servidor: `localhost`
> - Porta do PostgreSQL: `5432`
> - Usuário do PostgreSQL: `postgres`
> - Senha do PostgreSQL: `123456`
> - Banco de dados: `movies`

> ⚠️ O único SDK que você precisa baixar para acompanhar este material é o **SDK do .NET 8**. Não confunda **SDK** com **Runtime**: o Runtime apenas executa aplicações; o SDK permite criar, compilar, executar e publicar o projeto.

---

### 15.3.2 Antes de instalar

1. Salve seus trabalhos e feche terminais e instalações que estejam abertas.
2. Execute o **Windows Update** e reinicie o computador se houver atualização ou reinicialização pendente.
3. Confirme que sua conta pode autorizar a janela do **Controle de Conta de Usuário** do Windows.
4. Em **Configurações → Sistema → Sobre → Tipo de sistema**, confirme que aparece **sistema operacional de 64 bits, processador baseado em x64**. Este roteiro usa os instaladores x64 do .NET e do PostgreSQL.
5. Mantenha conexão com a internet durante a instalação do Visual Studio, porque o instalador baixa os componentes selecionados.

---

### 15.3.3 Instalando e configurando o Visual Studio Community

1. Abra <https://visualstudio.microsoft.com/pt-br/vs/community/>.
2. Clique no botão de download da edição **Community**.
3. Abra o arquivo baixado, normalmente chamado `VisualStudioSetup.exe` ou `vs_Community.exe`.
4. Quando o Windows perguntar se deseja permitir alterações no dispositivo, clique em **Sim**.
5. Leia os termos apresentados e clique em **Continuar** para abrir o **Visual Studio Installer**.
6. Na aba **Cargas de trabalho**, marque:
   - [x] **ASP.NET e desenvolvimento Web**
7. Abra a aba **Componentes individuais**, pesquise por `.NET SDK` e confirme que o componente de **SDK do .NET** está marcado. Dependendo da versão do Visual Studio Installer, o nome pode ser genérico ou podem aparecer componentes adicionais identificados como `.NET 8`; mantenha marcados os componentes recomendados pela carga de trabalho. A instalação independente do SDK 8 será feita e verificada na próxima seção.
8. Mantenha os demais componentes padrão da carga de trabalho e clique em **Instalar**.
9. Aguarde o download e a instalação. Reinicie o Windows se o instalador solicitar.
10. Abra o **Visual Studio Community**. O login com uma conta Microsoft pode ser feito, mas não é obrigatório para iniciar o projeto.
11. Na tela inicial, clique em **Criar um novo projeto** e pesquise por `API Web do ASP.NET Core`.
12. Confirme que esse template aparece. Você ainda não precisa criar o projeto; esta etapa serve apenas para validar a instalação.

Se o template não aparecer:

1. Feche o Visual Studio.
2. Abra **Visual Studio Installer** pelo menu Iniciar.
3. Na instalação Community, clique em **Modificar**.
4. Marque **ASP.NET e desenvolvimento Web**, confirme o componente de **SDK do .NET** e conclua a modificação.

---

### 15.3.4 Instalando e verificando o SDK do .NET 8

Mesmo que o Visual Studio tenha instalado componentes do .NET, faça esta verificação para garantir que o SDK esteja disponível também no terminal:

1. Abra <https://dotnet.microsoft.com/pt-br/download/dotnet/8.0>.
2. Localize a seção **Criar aplicativos — SDK**.
3. Na versão mais recente do **SDK 8.0**, localize a linha **Windows**.
4. Em **Instaladores**, clique em **x64**.
5. Abra o arquivo `.exe` baixado.
6. Autorize a instalação, clique em **Instalar** e aguarde a mensagem de conclusão.
7. Feche e abra novamente o Visual Studio, o PowerShell e qualquer Prompt de Comando que já estivesse aberto. Isso atualiza o ambiente usado pelos programas.
8. Abra um novo **PowerShell** e execute:

```bash
dotnet --list-sdks
dotnet --version
dotnet --info
dotnet new webapi --help
```

O comando `dotnet --list-sdks` precisa mostrar pelo menos uma linha iniciada por `8.0.`. Um exemplo de formato é:

```text
8.0.xxx [C:\Program Files\dotnet\sdk]
```

O número no lugar de `xxx` muda conforme as atualizações do SDK. Se houver outro SDK já instalado, `dotnet --version` poderá mostrar a versão mais nova selecionada; por isso, a confirmação decisiva do SDK 8 é a saída de `dotnet --list-sdks`.

Configure também o certificado HTTPS de desenvolvimento que será usado pela API:

```bash
dotnet dev-certs https --trust
```

Quando o Windows solicitar confirmação para confiar no certificado, aceite. Esse certificado é somente para o desenvolvimento local.

Se aparecer a mensagem de que `dotnet` não é reconhecido:

1. Feche todos os terminais e reinicie o Windows.
2. Abra um novo PowerShell e repita `dotnet --list-sdks`.
3. Se ainda falhar, execute novamente o instalador do SDK 8 e escolha **Reparar**, quando essa opção for apresentada.
4. Não prossiga até que uma versão `8.0.xxx` apareça na lista.

---

### 15.3.5 Instalando o PostgreSQL na porta 5432

Antes da instalação, abra o PowerShell e confira se a porta está livre:

```powershell
Get-NetTCPConnection -LocalPort 5432 -State Listen -ErrorAction SilentlyContinue
```

- Se o comando não mostrar nada, a porta `5432` está livre.
- Se mostrar uma linha, já existe um programa escutando nessa porta. **Não escolha outra porta e não prossiga com a instalação ainda.** Identifique e encerre ou reconfigure o programa conflitante com segurança. Depois, repita o comando e só continue quando ele não mostrar nenhuma linha, confirmando que a porta `5432` está livre.

Agora instale:

1. Abra <https://www.enterprisedb.com/downloads/postgres-postgresql-downloads>.
2. Escolha uma versão estável do PostgreSQL que ofereça instalador para **Windows x86-64**.
3. Baixe e abra o instalador `.exe` como administrador.
4. Na tela inicial do assistente, clique em **Next**.
5. Mantenha o diretório de instalação sugerido e clique em **Next**.
6. Na seleção de componentes, mantenha marcados:
   - [x] **PostgreSQL Server**
   - [x] **pgAdmin 4**
   - [x] **Command Line Tools**
   - [ ] **Stack Builder** é opcional e não será necessário neste projeto.
7. Mantenha o diretório de dados sugerido e clique em **Next**.
8. Na tela de senha do superusuário do banco:
   - **Password:** `123456`
   - **Retype password:** `123456`
9. Na tela **Port**, digite exatamente:

```text
5432
```

10. Na tela de localidade, mantenha **Default locale**.
11. Revise o resumo, clique em **Next** e aguarde a instalação.
12. Ao final, desmarque a abertura do **Stack Builder**, se essa opção aparecer, e clique em **Finish**.

> 🔒 A senha `123456` foi definida para manter todos os computadores da aula com a mesma configuração. Ela é fraca e deve ser usada **somente neste ambiente local de estudo**. Nunca use essa senha em produção ou em um banco exposto à internet.

---

### 15.3.6 Conferindo o serviço e configurando o pgAdmin 4

#### Conferir o serviço do PostgreSQL

1. Pressione `Win + R`.
2. Digite `services.msc` e pressione Enter.
3. Procure um serviço com nome semelhante a `postgresql-x64-XX`, em que `XX` representa a versão instalada.
4. Confirme:
   - **Status:** `Em execução`
   - **Tipo de inicialização:** `Automático`
5. Se estiver parado, clique com o botão direito no serviço e escolha **Iniciar**.

Você também pode conferir no PowerShell:

```powershell
Get-Service -Name "postgresql*"
Test-NetConnection -ComputerName localhost -Port 5432
```

No segundo comando, o resultado esperado é:

```text
TcpTestSucceeded : True
```

#### Abrir e conectar o pgAdmin 4

1. Abra **pgAdmin 4** pelo menu Iniciar.
2. Se ele solicitar uma **senha mestra**, lembre-se: essa senha protege as credenciais salvas dentro do pgAdmin e não é a senha do banco. Para este computador de estudo, você pode usar `123456` também para não se confundir.
3. No painel esquerdo, expanda **Servers**.
4. Se o servidor criado pelo instalador já aparecer, clique nele e informe a senha `123456`.
5. Se nenhum servidor aparecer, clique com o botão direito em **Servers → Register → Server...** e preencha:

| Aba | Campo | Valor |
|---|---|---|
| **General** | Name | `PostgreSQL local` |
| **Connection** | Host name/address | `localhost` |
| **Connection** | Port | `5432` |
| **Connection** | Maintenance database | `postgres` |
| **Connection** | Username | `postgres` |
| **Connection** | Password | `123456` |
| **Connection** | Save password | marcado |

Clique em **Save**. O servidor deve aparecer conectado no painel esquerdo.

#### Criar o banco `movies`

1. Expanda o servidor conectado.
2. Clique com o botão direito em **Databases → Create → Database...**.
3. Em **Database**, digite `movies`.
4. Em **Owner**, selecione `postgres`.
5. Clique em **Save**.
6. Expanda **Databases** e confirme que `movies` aparece na lista. Se necessário, clique com o botão direito em **Databases → Refresh**.
7. Clique no banco `movies`, abra **Tools → Query Tool**, digite e execute:

```sql
SHOW port;
SELECT current_user, current_database();
```

Os resultados esperados são:

- porta: `5432`
- usuário atual: `postgres`
- banco atual: `movies`

A connection string que será usada neste projeto é:

```text
Host=localhost;Port=5432;Database=movies;Username=postgres;Password=123456
```

> 💡 A porta `5432` é a porta do **PostgreSQL**. Ela não é a porta HTTPS da Movies.API. A API receberá outra porta local quando for executada pelo Visual Studio ou pelo comando `dotnet run`.

---

### 15.3.7 Verificação final da preparação

Antes de seguir, confirme do zero:

- [ ] O Visual Studio Community abre normalmente.
- [ ] O template **API Web do ASP.NET Core** aparece no Visual Studio.
- [ ] A carga de trabalho **ASP.NET e desenvolvimento Web** está instalada.
- [ ] `dotnet --list-sdks` mostra uma versão `8.0.xxx`.
- [ ] `dotnet new webapi --help` funciona.
- [ ] O certificado HTTPS de desenvolvimento foi confiado.
- [ ] O serviço do PostgreSQL está em execução.
- [ ] `Test-NetConnection localhost -Port 5432` mostra `TcpTestSucceeded : True`.
- [ ] O pgAdmin conecta em `localhost:5432` com usuário `postgres` e senha `123456`.
- [ ] O banco `movies` existe e pertence ao usuário `postgres`.
- [ ] `SHOW port;` retorna `5432`.

Se todos os itens estiverem corretos, o computador está pronto para criar e executar a Movies.API.

---

Antes de começar, confirme:

- [ ] **.NET 8 SDK** instalado: `dotnet --version` → `8.x.x`
- [ ] **PostgreSQL** rodando na porta `5432`
- [ ] usuário `postgres` / senha `123456`
- [ ] **Visual Studio 2022+** ou **VS Code com C# Dev Kit**
- [ ] **CLI do EF Core**:
  ```bash
  dotnet tool install --global dotnet-ef
  ```

---

## 15.4 Criando o projeto

Você pode criar o projeto de **duas maneiras**: pela **interface do Visual Studio** (mais visual, recomendado para iniciantes) ou pelo **terminal** (mais rápido, recomendado quando você já dominar). Escolha a que preferir — o resultado é o mesmo.

---

### 🅰️ Opção A — Pelo Visual Studio (recomendado para iniciantes)

#### Passo 1 — Abrir o Visual Studio e criar projeto

1. Abra o **Visual Studio 2022** (ou superior).
2. Na tela inicial, clique em **"Criar um novo projeto"**.

#### Passo 2 — Escolher o template "API Web do ASP.NET Core"

Na janela **"Criar um novo projeto"**:

1. No campo de busca, digite: `API Web do ASP.NET Core` (ou `ASP.NET Core Web API` se estiver em inglês).
2. Selecione o template **"API Web do ASP.NET Core"** (ícone azul com um globo). **Cuidado** para não confundir com:
   - ❌ **"ASP.NET Core API Web (native AOT)"** — versão avançada, não use.
   - ❌ **"Aplicativo Web ASP.NET Core (Razor Pages)"** — não é API.
   - ❌ **"App de Inicialização Aspire"** — projeto fullstack, não é o que queremos.
3. Confirme que os filtros estão marcados como **C#**, **API**, **Web API**.
4. Clique em **"Próximo"**.

> 💡 Veja na imagem: é o template com os marcadores **"C# / Linux / macOS / Windows / API / Nuvem / Serviço / Web / Web API"**.

#### Passo 3 — Configurar o projeto

Na tela **"Configurar seu novo projeto"**:

| Campo | Valor |
|---|---|
| **Nome do projeto** | `Movies.API` |
| **Local** | escolha uma pasta (ex.: `C:\dev\backend`) |
| **Nome da solução** | `Movies.API` (deixe igual) |
| **Colocar solução e projeto no mesmo diretório** | ✅ marcado (opcional, mas simplifica) |

Clique em **"Próximo"**.

#### Passo 4 — Informações adicionais

Na tela **"Informações adicionais"**, preencha **exatamente assim**:

| Campo | Valor |
|---|---|
| **Estrutura (Framework)** | **.NET 8.0 (Suporte de longo prazo)** |
| **Tipo de autenticação** | **Nenhum** (vamos implementar JWT manualmente) |
| **Configurar para HTTPS** | ✅ marcado |
| **Habilitar suporte a OpenAPI** | ✅ marcado (gera Swagger) |
| **Não usar instruções de nível superior** | ❌ desmarcado |
| **Usar controladores (desmarque para usar APIs mínimas)** | ✅ **MARCADO** (importantíssimo!) |
| **Inscrever-se no contêiner do .NET Aspire** | ❌ desmarcado |
| **Habilitar contêiner do Docker** | ❌ desmarcado |

Clique em **"Criar"**.

> ⚠️ **Atenção ao "Usar controladores"**: se ficar **desmarcado**, o projeto usará Minimal APIs e a apostila não vai funcionar como descrito. **Marque essa opção.**

#### Passo 5 — Limpar arquivos de exemplo

O Visual Studio cria alguns arquivos de demonstração. Apague-os:

- `Controllers/WeatherForecastController.cs` → clique com o botão direito → **Excluir**.
- `WeatherForecast.cs` (na raiz) → **Excluir**.

#### Passo 6 — Instalar os pacotes NuGet pela interface

1. No **Solution Explorer**, clique com o botão direito no projeto **Movies.API** → **"Gerenciar Pacotes NuGet..."**.
2. Vá na aba **"Procurar"**.
3. Para cada pacote abaixo:
   - Digite o nome no campo de busca.
   - Selecione o pacote oficial (autor: **Microsoft** ou **Npgsql**).
   - **No painel da direita**, troque a versão para a indicada.
   - Clique em **"Instalar"** e aceite os termos quando pedir.

| Pacote                                          | Versão     |
| ----------------------------------------------- | ---------- |
| `Microsoft.AspNetCore.Authentication.JwtBearer` | **8.0.10** |
| `Microsoft.EntityFrameworkCore`                 | **9.0.8**  |
| `Microsoft.EntityFrameworkCore.Design`          | **9.0.8**  |
| `Microsoft.IdentityModel.Tokens`                | **8.14.0** |
| `Npgsql.EntityFrameworkCore.PostgreSQL`         | **9.0.4**  |
| `Swashbuckle.AspNetCore`                        | **6.6.2**  |
| `System.IdentityModel.Tokens.Jwt`               | **8.14.0** |

> 💡 Para conferir tudo depois, abra o arquivo `Movies.API.csproj` (clique duplo no nome do projeto) e veja a lista de `<PackageReference>`.

#### Passo 7 — Rodar para testar

Pressione **F5** (ou clique no botão verde ▶ **"https"** no topo). O Visual Studio deve:

1. Compilar o projeto.
2. Abrir o navegador na URL do Swagger (`https://localhost:7XXX/swagger`).

Se aparecer a tela do Swagger, está tudo certo. Pare a execução com **Shift+F5** e siga para o Passo 1 do tutorial (criar `Controllers/HealthCheck.cs`).

---

### 🅱️ Opção B — Pelo terminal (CLI)

Mais rápido, ideal depois que você já estiver acostumado:

```bash
mkdir backend
cd backend
dotnet new webapi -n Movies.API --use-controllers
cd Movies.API
```

Apague os arquivos `WeatherForecastController.cs` e `WeatherForecast.cs` que vêm de exemplo.

#### Instalando os pacotes via CLI

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer --version 8.0.10
dotnet add package Microsoft.EntityFrameworkCore --version 9.0.8
dotnet add package Microsoft.EntityFrameworkCore.Design --version 9.0.8
dotnet add package Microsoft.IdentityModel.Tokens --version 8.14.0
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 9.0.4
dotnet add package Swashbuckle.AspNetCore --version 6.6.2
dotnet add package System.IdentityModel.Tokens.Jwt --version 8.14.0
```

> 💡 **Dica híbrida**: você também pode criar o projeto pela interface (Opção A) e instalar os pacotes pelo terminal (rodando os comandos acima dentro da pasta do projeto, no Visual Studio acesse pela aba **"Exibir → Terminal"**).

---

### 🛠️ Criando pastas no Visual Studio

Para criar as pastas que vamos usar (`Models`, `Controllers`, `Services`, etc.):

1. **Solution Explorer** → clique com o botão direito no projeto.
2. **Adicionar → Nova Pasta**.
3. Digite o nome (ex.: `Models`) e pressione Enter.

Para criar **subpastas** (ex.: `Request/Movies`):

1. Clique direito na pasta `Request` → **Adicionar → Nova Pasta** → `Movies`.

Para criar **classes** dentro de uma pasta:

1. Clique direito na pasta → **Adicionar → Classe...**
2. Digite o nome (ex.: `Movie.cs`) e clique em **Adicionar**.

> 💡 **Atalho mais rápido**: clique direito → **Adicionar → Novo Item...** → escolha **"Classe"** → digite o nome → **Adicionar**.



### Estrutura final que vamos construir

```
Movies.API/
├── Authentication/      ← JwtSettings.cs, JwtAuthManager.cs
├── Controllers/         ← HealthCheck, Movie, Login, User
├── DatabaseContext/     ← DataContext.cs
├── Encrypt/             ← PasswordEncryptor.cs
├── Interfaces/Repository/ ← IRepositoryMovie, IRepositoryUser
├── Mappings/            ← MovieMap.cs, UserMap.cs
├── Migrations/          ← gerado pelo EF Core
├── Models/              ← Movie.cs, User.cs, Login.cs
├── Request/
│   ├── Authentication/  ← AuthRequest.cs
│   ├── Movies/          ← MovieCreateRequest.cs, MovieUpdateRequest.cs
│   └── Users/           ← UserCreateRequest.cs, UserUpdateRequest.cs
├── Services/            ← MovieService.cs, UserService.cs
├── appsettings.json
└── Program.cs           ← ÚLTIMO!
```

---

## ✅ Passo 1 — `Controllers/HealthCheck.cs`

> **O primeiro arquivo do projeto deve ser o HealthCheck.** Antes de qualquer Model, Service ou Controller de negócio, criamos um endpoint **simples e independente** que prova que a aplicação está de pé. Se este endpoint não responder, nada mais vai funcionar — então melhor descobrir agora.

Crie a pasta `Controllers/` (se ainda não existir) e dentro dela o arquivo `HealthCheck.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace Movies.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public class HealthCheck : ControllerBase
{
    [HttpGet]
    public IActionResult Check()
    {
        return Ok("The API is working!");
    }
}
```

| Método | Rota | Resposta |
|---|---|---|
| GET | `/api/HealthCheck` | `200 OK` `"The API is working!"` |

### 🧪 Teste **agora**, antes de continuar

1. Rode com **F5** (Visual Studio) ou `dotnet run` (CLI).
2. Abra `https://localhost:PORTA/swagger`.
3. Expanda **HealthCheck → GET `/api/HealthCheck`** → **Try it out** → **Execute**.
4. Esperado: status `200` com corpo `"The API is working!"`.

Funcionou? Pare a execução (**Shift+F5** ou Ctrl+C) e siga para o Passo 2.

> 💡 **Por que isso é o primeiro arquivo?** Porque ele **não depende de nada** — sem banco, sem JWT, sem Models. Se quebrar aqui, é problema de **infraestrutura** (template errado, pacote faltando, porta ocupada). Resolver isso agora é muito mais fácil do que descobrir lá no fim, depois de já ter escrito 20 arquivos.

---

## ✅ Passo 2 — `Models/Movie.cs`

> Models são apenas representações de dados — não dependem de nada além da própria linguagem.

```csharp
namespace Movies.API.Models;

public class Movie
{
    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public string PosterUrl { get; set; } = string.Empty;
    public string Overview { get; set; } = string.Empty;

    public Movie() { }

    public Movie(string title, string posterUrl, string overview)
    {
        Title = title;
        PosterUrl = posterUrl;
        Overview = overview;
    }
}
```

> 💡 O **construtor sem parâmetros** é exigido pelo EF Core para reconstruir objetos vindos do banco.

---

## ✅ Passo 3 — `Models/User.cs`

```csharp
namespace Movies.API.Models;

public class User
{
    public int Id { get; set; }
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;

    public User() { }

    public User(string username, string password)
    {
        Username = username;
        Password = password;
    }
}
```

> 🔒 O `Password` aqui guardará o **hash** da senha — nunca o texto puro.

---

## ✅ Passo 4 — `Models/Login.cs`

```csharp
namespace Movies.API.Models;

public class Login
{
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

> POCO simples para representar credenciais (não vai para o banco).

---

## ✅ Passo 5 — `Request/Movies/MovieCreateRequest.cs`

> **DTOs (Request)** existem para separar o que **entra pela API** do que vai para o **banco**.

```csharp
namespace Movies.API.Request.Movies;

public class MovieCreateRequest
{
    public string Title { get; set; } = string.Empty;
    public string PosterUrl { get; set; } = string.Empty;
    public string Overview { get; set; } = string.Empty;
}
```

---

## ✅ Passo 6 — `Request/Movies/MovieUpdateRequest.cs`

```csharp
namespace Movies.API.Request.Movies;

public class MovieUpdateRequest
{
    public string Title { get; set; } = string.Empty;
    public string PosterUrl { get; set; } = string.Empty;
    public string Overview { get; set; } = string.Empty;
}
```

> ⚠️ Sim, neste projeto Create e Update são iguais. Em projetos reais, podem ser diferentes (Update permite só alguns campos, por exemplo).

---

## ✅ Passo 7 — `Request/Users/UserCreateRequest.cs`

```csharp
namespace Movies.API.Request.Users;

public class UserCreateRequest
{
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

---

## ✅ Passo 8 — `Request/Users/UserUpdateRequest.cs`

```csharp
namespace Movies.API.Request.Users;

public class UserUpdateRequest
{
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

---

## ✅ Passo 9 — `Request/Authentication/AuthRequest.cs`

```csharp
namespace Movies.API.Request.Login;   // ← namespace é "Login", não "Authentication"!

public class AuthRequest
{
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

> ⚠️ **Atenção peculiar**: o arquivo está em `Request/Authentication/`, mas o **namespace** é `Movies.API.Request.Login`. Isso é importante para o `LoginController` importar corretamente.

---

## ✅ Passo 10 — `Encrypt/PasswordEncryptor.cs`

> **Criptografa senhas com SHA-256 + Base64.** Sempre antes de salvar, sempre antes de comparar.

```csharp
using System.Security.Cryptography;
using System.Text;

namespace Movies.API.Encrypt;

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

    public static bool VerifyPassword(string password, string hashedPassword)
    {
        if (string.IsNullOrEmpty(password) || string.IsNullOrEmpty(hashedPassword))
            return false;

        var encryptedInput = EncryptPassword(password);
        return encryptedInput == hashedPassword;
    }
}
```

---

## ✅ Passo 11 — `Authentication/JwtSettings.cs`

> POCO que **espelha** a seção `JwtSettings` do `appsettings.json`.

```csharp
namespace Movies.API.Authentication;

public class JwtSettings
{
    public string Key { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public double DurationMinutes { get; set; }
}
```

---

## ✅ Passo 12 — `appsettings.json`

> Configurações da aplicação. **Estrutura precisa bater** com `JwtSettings`.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=movies;Username=postgres;Password=123456"
  },
  "JwtSettings": {
    "Key": "YourStrongSecretKeyHere1234567890",
    "Issuer": "Movies.API",
    "Audience": "Movies.API",
    "DurationMinutes": 60
  }
}
```

> ⚠️ A `Key` precisa ter **pelo menos 32 caracteres**. Em produção, use uma chave aleatória forte.

---

## ✅ Passo 13 — `Mappings/MovieMap.cs`

> Diz ao EF Core **como mapear** `Movie` para a tabela `movies`.

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using Movies.API.Models;

namespace Movies.API.Mappings;

public class MovieMap : IEntityTypeConfiguration<Movie>
{
    public void Configure(EntityTypeBuilder<Movie> builder)
    {
        builder.ToTable("movies");
        builder.HasKey(x => x.Id);

        builder.Property(x => x.Id).HasColumnName("id");
        builder.Property(x => x.Title)
               .HasColumnName("title")
               .HasMaxLength(100)
               .IsRequired();
        builder.Property(x => x.PosterUrl)
               .HasColumnName("poster_url")
               .HasMaxLength(255)
               .IsRequired();
        builder.Property(x => x.Overview)
               .HasColumnName("overview")
               .IsRequired();
    }
}
```

---

## ✅ Passo 14 — `Mappings/UserMap.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using Movies.API.Models;

namespace Movies.API.Mappings;

public class UserMap : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("users");
        builder.HasKey(x => x.Id);

        builder.Property(x => x.Id).HasColumnName("id");
        builder.Property(x => x.Username)
               .HasColumnName("username")
               .HasMaxLength(50)
               .IsRequired();
        builder.Property(x => x.Password)
               .HasColumnName("password")
               .HasMaxLength(100)
               .IsRequired();
    }
}
```

---

## ✅ Passo 15 — `Authentication/JwtAuthManager.cs`

> Gera o JWT assinado quando o login dá certo.

```csharp
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

namespace Movies.API.Authentication;

public static class JwtAuthManager
{
    public static string GenerateToken(string userName)
    {
        IConfigurationRoot configuration = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json")
            .Build();

        var jwtSettings = configuration.GetSection("JwtSettings").Get<JwtSettings>()
            ?? throw new InvalidOperationException("JwtSettings não configurado");

        if (string.IsNullOrEmpty(jwtSettings.Key))
            throw new InvalidOperationException("JwtSettings.Key vazia");

        var key = Encoding.ASCII.GetBytes(jwtSettings.Key);

        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, userName)
        };

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

        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }
}
```

---

## ✅ Passo 16 — `DatabaseContext/DataContext.cs`

> O **coração da persistência**. Conecta a aplicação ao PostgreSQL.

```csharp
using Microsoft.EntityFrameworkCore;
using Movies.API.Mappings;
using Movies.API.Models;

namespace Movies.API.DatabaseContext;

public class DataContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Movie> Movies { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        IConfigurationRoot configuration = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json")
            .Build();

        var connectionString = configuration.GetConnectionString("DefaultConnection");
        optionsBuilder.UseNpgsql(connectionString);
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfiguration(new UserMap());
        modelBuilder.ApplyConfiguration(new MovieMap());
    }
}
```

---

## ✅ Passo 17 — Migrations

> Agora que `DataContext` + Mappings + Models existem, podemos gerar a migration.

```bash
dotnet ef migrations add DatabaseCreation
dotnet ef database update
```

Verifique no **pgAdmin**: as tabelas `movies` e `users` devem aparecer no banco `movies`.

> Se der erro:
> - PostgreSQL não está rodando? → Inicie o serviço.
> - Banco `movies` não existe? → Crie no pgAdmin.
> - Senha errada? → Confira o `appsettings.json`.

---

## ✅ Passo 18 — `Interfaces/Repository/IRepositoryMovie.cs`

> **Contrato** que o `MovieService` vai implementar.

```csharp
using Movies.API.Models;
using Movies.API.Request.Movies;

namespace Movies.API.Interfaces.Repository;

public interface IRepositoryMovie
{
    bool Create(MovieCreateRequest movie);
    Movie? GetById(int id);
    bool Update(int id, MovieUpdateRequest movie);
    bool Delete(int id);
    List<Movie> GetAll();
}
```

---

## ✅ Passo 19 — `Interfaces/Repository/IRepositoryUser.cs`

```csharp
using Movies.API.Models;
using Movies.API.Request.Users;

namespace Movies.API.Interfaces.Repository;

public interface IRepositoryUser
{
    bool Create(UserCreateRequest user);
    User? GetById(int id);
    bool Update(int id, UserUpdateRequest user);
    bool Delete(int id);
    IEnumerable<User> GetAll();
}
```

---

## ✅ Passo 20 — `Services/MovieService.cs`

> Implementa o contrato e fala com o banco via `DataContext`.

```csharp
using Microsoft.EntityFrameworkCore;
using Movies.API.DatabaseContext;
using Movies.API.Interfaces.Repository;
using Movies.API.Models;
using Movies.API.Request.Movies;

namespace Movies.API.Services;

public class MovieService : IRepositoryMovie
{
    public bool Create(MovieCreateRequest request)
    {
        using var connection = new DataContext();
        try
        {
            var movie = new Movie(request.Title, request.PosterUrl, request.Overview);
            connection.Movies.Add(movie);
            connection.SaveChanges();
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return false;
        }
    }

    public Movie? GetById(int id)
    {
        using var connection = new DataContext();
        try
        {
            return connection.Movies.AsNoTracking().FirstOrDefault(m => m.Id == id);
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return null;
        }
    }

    public bool Update(int id, MovieUpdateRequest request)
    {
        using var connection = new DataContext();
        try
        {
            var movie = connection.Movies.FirstOrDefault(m => m.Id == id);
            if (movie == null) return false;

            movie.Title = request.Title;
            movie.PosterUrl = request.PosterUrl;
            movie.Overview = request.Overview;
            connection.SaveChanges();
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return false;
        }
    }

    public bool Delete(int id)
    {
        using var connection = new DataContext();
        try
        {
            var movie = connection.Movies.FirstOrDefault(m => m.Id == id);
            if (movie == null) return false;

            connection.Movies.Remove(movie);
            connection.SaveChanges();
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return false;
        }
    }

    public List<Movie> GetAll()
    {
        using var connection = new DataContext();
        try
        {
            return connection.Movies.AsNoTracking().ToList();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return new List<Movie>();
        }
    }
}
```

> 💡 **Padrões aplicados**:
> - `using var` → fecha conexão automaticamente.
> - `AsNoTracking()` em leituras → mais performance.
> - `try/catch` → API não quebra com exceções.

---

## ✅ Passo 21 — `Services/UserService.cs`

> A **diferença chave** em relação ao `MovieService`: senha é **encriptada** antes de salvar.

```csharp
using Microsoft.EntityFrameworkCore;
using Movies.API.DatabaseContext;
using Movies.API.Encrypt;
using Movies.API.Interfaces.Repository;
using Movies.API.Models;
using Movies.API.Request.Users;

namespace Movies.API.Services;

public class UserService : IRepositoryUser
{
    public bool Create(UserCreateRequest request)
    {
        using var connection = new DataContext();
        try
        {
            var hash = PasswordEncryptor.EncryptPassword(request.Password);
            var user = new User(request.Username, hash);
            connection.Users.Add(user);
            connection.SaveChanges();
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return false;
        }
    }

    public User? GetById(int id)
    {
        using var connection = new DataContext();
        try
        {
            return connection.Users.AsNoTracking().FirstOrDefault(u => u.Id == id);
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return null;
        }
    }

    public bool Update(int id, UserUpdateRequest request)
    {
        using var connection = new DataContext();
        try
        {
            var user = connection.Users.FirstOrDefault(u => u.Id == id);
            if (user == null) return false;

            user.Username = request.Username;
            user.Password = PasswordEncryptor.EncryptPassword(request.Password);
            connection.SaveChanges();
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return false;
        }
    }

    public bool Delete(int id)
    {
        using var connection = new DataContext();
        try
        {
            var user = connection.Users.FirstOrDefault(u => u.Id == id);
            if (user == null) return false;

            connection.Users.Remove(user);
            connection.SaveChanges();
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return false;
        }
    }

    public IEnumerable<User> GetAll()
    {
        using var connection = new DataContext();
        try
        {
            return connection.Users.AsNoTracking().ToList();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return new List<User>();
        }
    }
}
```

---

## ✅ Passo 22 — `Controllers/MovieController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using Movies.API.Request.Movies;
using Movies.API.Services;

namespace Movies.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public class MovieController : ControllerBase
{
    private readonly MovieService _service = new();

    [HttpPost]
    public IActionResult Create([FromBody] MovieCreateRequest request)
    {
        return _service.Create(request)
            ? Ok("Movie created with success!")
            : BadRequest("Failed to create movie");
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var movie = _service.GetById(id);
        return movie == null ? NotFound() : Ok(movie);
    }

    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] MovieUpdateRequest request)
    {
        return _service.Update(id, request)
            ? Ok("Updated!")
            : BadRequest("Failed to update");
    }

    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        return _service.Delete(id) ? Ok("Deleted!") : BadRequest("Failed to delete");
    }

    [HttpGet("get-all")]
    public IActionResult GetAll() => Ok(_service.GetAll());
}
```

> ℹ️ Note que **`MovieController` não tem `[Authorize]`** — está aberto.

---

## ✅ Passo 23 — `Controllers/LoginController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Movies.API.Authentication;
using Movies.API.DatabaseContext;
using Movies.API.Encrypt;
using Movies.API.Request.Login;

namespace Movies.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public class LoginController : ControllerBase
{
    [HttpPost]
    public IActionResult Login([FromBody] AuthRequest request)
    {
        using var connection = new DataContext();

        var hash = PasswordEncryptor.EncryptPassword(request.Password);

        var user = connection.Users
            .AsNoTracking()
            .FirstOrDefault(u => u.Username == request.Username && u.Password == hash);

        if (user == null)
            return BadRequest("Login failed!");

        var token = JwtAuthManager.GenerateToken(user.Username);
        return Ok($"Login successfull! Jwt: {token}");
    }
}
```

---

## ✅ Passo 24 — `Controllers/UserController.cs`

> 🔒 **Todo o controller é protegido por `[Authorize]`** — exige token JWT válido.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Movies.API.Request.Users;
using Movies.API.Services;

namespace Movies.API.Controllers;

[ApiController]
[Route("api/[controller]")]
[Authorize]
public class UserController : ControllerBase
{
    private readonly UserService _service = new();

    [HttpPost]
    public IActionResult Create([FromBody] UserCreateRequest request)
    {
        return _service.Create(request)
            ? Ok("User created with success!")
            : BadRequest("Failed");
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var user = _service.GetById(id);
        return user == null ? NotFound() : Ok(user);
    }

    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] UserUpdateRequest request)
    {
        return _service.Update(id, request) ? Ok("Updated!") : BadRequest("Failed");
    }

    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        return _service.Delete(id) ? Ok("Deleted!") : BadRequest("Failed");
    }

    [HttpGet("get-all")]
    public IActionResult GetAll() => Ok(_service.GetAll());
}
```

---

## ✅ Passo 25 — `Program.cs` (POR ÚLTIMO)

> **Depende de TUDO acima.** Por isso é o último.

```csharp
using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi.Models;
using Movies.API.Authentication;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();

// 1) JWT
var jwtSettings = builder.Configuration.GetSection("JwtSettings").Get<JwtSettings>()
    ?? throw new InvalidOperationException("JwtSettings não configurado");

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

// 2) Swagger com suporte a Bearer
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Movies.API", Version = "v1" });

    var securityScheme = new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Insira: Bearer {seu_token}"
    };
    c.AddSecurityDefinition("Bearer", securityScheme);

    var securityRequirement = new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    };
    c.AddSecurityRequirement(securityRequirement);
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// 3) ORDEM OBRIGATÓRIA: Authentication ANTES de Authorization
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

---

## 15.5 Rodando a aplicação

```bash
dotnet run
```

Acesse: <https://localhost:PORTA/swagger>

---

## 15.6 Testando o fluxo completo

### Etapa 1 — Verifique o HealthCheck

`GET /api/HealthCheck` → deve responder `"The API is working!"`.

### Etapa 2 — Crie um filme (público)

`POST /api/Movie`:
```json
{
  "title": "Matrix",
  "posterUrl": "https://exemplo.com/matrix.jpg",
  "overview": "Filme sobre realidade virtual"
}
```

`GET /api/Movie/get-all` → você verá o filme.

### Etapa 3 — Crie um usuário (problema do "ovo e galinha")

`POST /api/User` exige autenticação. Mas você ainda não tem usuários para fazer login!

**Solução temporária**: insira um usuário manualmente no banco via pgAdmin:

```sql
-- Senha "123" hasheada em SHA256+Base64 = "pmWkWSBCL51Bfkhn79xPuKBKHz//H6B+mY6G9/eieuM="
INSERT INTO users (username, password)
VALUES ('admin', 'pmWkWSBCL51Bfkhn79xPuKBKHz//H6B+mY6G9/eieuM=');
```

> Em projetos reais, costuma-se **liberar temporariamente** o `POST /api/User` (com `[AllowAnonymous]`) ou criar um endpoint de **seed** público para o primeiro usuário.

### Etapa 4 — Faça login

`POST /api/Login`:
```json
{ "username": "admin", "password": "123" }
```

Resposta:
```
Login successfull! Jwt: eyJhbGciOiJIUzI1NiIs...
```

### Etapa 5 — Use o token

1. No Swagger, clique em **🔒 Authorize** no topo.
2. Cole: `Bearer eyJhbGciOiJIUzI1NiIs...` (com a palavra Bearer + espaço + token).
3. Agora teste `GET /api/User/get-all` → deve listar os usuários.

Sem o token: `401 Unauthorized`. Com token expirado/inválido: também `401`.

---

## 15.7 Resumo da ordem de criação (cola rápida)

| # | Arquivo | Depende de |
|---|---|---|
| 1 | `Controllers/HealthCheck.cs` | nada (teste de saúde — **crie e teste primeiro**) |
| 2 | `Models/Movie.cs` | nada |
| 3 | `Models/User.cs` | nada |
| 4 | `Models/Login.cs` | nada |
| 5 | `Request/Movies/MovieCreateRequest.cs` | nada |
| 6 | `Request/Movies/MovieUpdateRequest.cs` | nada |
| 7 | `Request/Users/UserCreateRequest.cs` | nada |
| 8 | `Request/Users/UserUpdateRequest.cs` | nada |
| 9 | `Request/Authentication/AuthRequest.cs` | nada |
| 10 | `Encrypt/PasswordEncryptor.cs` | nada |
| 11 | `Authentication/JwtSettings.cs` | nada |
| 12 | `appsettings.json` | JwtSettings (estrutura) |
| 13 | `Mappings/MovieMap.cs` | Movie |
| 14 | `Mappings/UserMap.cs` | User |
| 15 | `Authentication/JwtAuthManager.cs` | JwtSettings |
| 16 | `DatabaseContext/DataContext.cs` | Models + Mappings |
| 17 | `Migrations/` (gerado) | DataContext |
| 18 | `Interfaces/Repository/IRepositoryMovie.cs` | Movie + Requests |
| 19 | `Interfaces/Repository/IRepositoryUser.cs` | User + Requests |
| 20 | `Services/MovieService.cs` | tudo acima de filmes |
| 21 | `Services/UserService.cs` | tudo + PasswordEncryptor |
| 22 | `Controllers/MovieController.cs` | MovieService |
| 23 | `Controllers/LoginController.cs` | DataContext + Encryptor + JwtManager + AuthRequest |
| 24 | `Controllers/UserController.cs` | UserService + JWT |
| 25 | `Program.cs` | **TUDO** |

---

## 15.8 Mapa de dependências (visual)

```
Models ──► Mappings ──► DataContext ──► Migrations
   │                          │
   │                          └────► Services ──► Controllers
   │                                    │              │
Requests ──────────────────────────────►│              │
                                                       │
JwtSettings ──► JwtAuthManager ────────────────────────┤
                                                       │
PasswordEncryptor ─────────────────────────────────────┤
                                                       ▼
                                                   Program.cs
```

---

## 15.9 Erros comuns e soluções

| Erro | Causa provável | Solução |
|---|---|---|
| `Connection refused` ao iniciar | PostgreSQL não está rodando | Inicie o serviço |
| `database "movies" does not exist` | Banco não criado | Crie via pgAdmin |
| `password authentication failed` | Senha errada | Confira `appsettings.json` |
| `No DbContext was found` no `dotnet ef` | Comando rodado da pasta errada | Rode da pasta do `.csproj` |
| Swagger sem botão "Authorize" | `AddSecurityDefinition` esquecido | Reveja o `Program.cs` |
| `401 Unauthorized` mesmo com token | Token sem `Bearer` antes / chave diferente | Cheque o formato |
| `IDX10720: Unable to create KeyedHashAlgorithm` | Chave JWT muito curta | Use ≥ 32 caracteres |

---

## 15.10 Próximos passos

Você acabou de construir uma API profissional. Daqui em diante, evolua-a:

1. **Validação de modelos** com Data Annotations (Cap. 10).
2. **Refatorar Services para usar DI** (`builder.Services.AddScoped<IRepositoryMovie, MovieService>()`).
3. **AutoMapper** para mapear DTO ↔ Model automaticamente.
4. **Migrar para BCrypt** em vez de SHA-256 puro.
5. **Adicionar paginação** em `GetAll`.
6. **Testes automatizados** com xUnit.
7. **Deploy** no Azure / AWS / Render.
8. **Frontend** (Vue, React, Angular) consumindo a API.

---

## 15.11 Resumo do capítulo

- Construímos uma API completa do zero ao funcional.
- Seguimos a **ordem de criação por dependência** — Controllers e `Program.cs` por último.
- Aplicamos **EF Core + PostgreSQL**, **JWT + Swagger** e **camadas (Model, Mapping, Repository, Service, Controller)**.
- Vimos como testar tudo no Swagger.

---

## 🎓 Parabéns!

Você concluiu a apostila. Saiu do **zero absoluto** e chegou a uma **API profissional, autenticada, com banco de dados real e documentação interativa**.

Daqui pra frente: **construa, erre, conserte, repita**. É assim que se vira programador de verdade.

---

⬅️ **Voltar ao[Sumário](3-Apostilas/backend/01_csharp_dotnet/dotnet_8/00-Index.md))**

---
[[1-index|Voltar para o index]]

