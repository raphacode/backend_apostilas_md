# Capítulo 13 — Entity Framework Core e PostgreSQL

> Até aqui, nossas APIs guardavam dados **na memória**. Quando o programa parava, tudo sumia. Agora vamos aprender a **persistir de verdade** — em um banco de dados PostgreSQL — usando **Entity Framework Core (EF Core)**.

> 💡 Este capítulo prepara o terreno para o **Capítulo 15**, onde vamos construir uma API completa de filmes com PostgreSQL e JWT.

---

## 13.1 O que é um banco de dados relacional?

Um **banco de dados relacional** organiza informações em **tabelas** — como planilhas conectadas entre si.

Exemplo de uma tabela `movies`:

| id | title | poster_url | overview |
|----|-------|------------|----------|
| 1  | Matrix | http://... | Filme sobre... |
| 2  | Interstellar | http://... | Astronautas... |

Linguagem usada: **SQL** (Structured Query Language).

```sql
SELECT * FROM movies WHERE id = 1;
INSERT INTO movies (title, ...) VALUES ('Matrix', ...);
```

### Bancos relacionais populares

- **PostgreSQL** — gratuito, robusto, **vamos usar este**.
- **SQL Server** — da Microsoft, muito comum em ambientes .NET.
- **MySQL** / **MariaDB** — gratuitos, muito comuns na web.
- **SQLite** — banco em **um único arquivo**, ótimo para protótipos.

---

## 13.2 O que é um ORM?

**ORM** = **Object-Relational Mapper** (Mapeador Objeto-Relacional).

> Um ORM permite que você **trabalhe com objetos C#** em vez de escrever SQL na mão. Ele traduz seus objetos em comandos SQL e vice-versa.

Em vez de:

```sql
INSERT INTO movies (title, poster_url, overview) VALUES (...);
```

Você escreve:

```csharp
context.Movies.Add(new Movie { Title = "Matrix", ... });
context.SaveChanges();
```

O ORM gera o SQL por você. **Entity Framework Core** é o ORM oficial do .NET.

### Vantagens

- Menos SQL escrito à mão = menos erros de digitação.
- Migrations: **versiona o banco** como se fosse código.
- Trabalha com objetos C# **fortemente tipados**.
- Mudar de banco (SQL Server → PostgreSQL) exige pouquíssimo esforço.

### Desvantagens

- Consultas muito complexas podem ficar lentas se mal escritas.
- Curva de aprendizado inicial.
- Pode "esconder" o que está acontecendo no banco.

---

## 13.3 Instalando o PostgreSQL

### Windows

1. Baixe em <https://www.postgresql.org/download/windows/>.
2. Durante a instalação:
   - Defina uma **senha** para o usuário `postgres` (ex.: `123456`).
   - Aceite a porta padrão `5432` — ou mude para `5433` (vamos usar `5433` no projeto Movies.API).
3. Instale a ferramenta **pgAdmin** que vem junto — é a interface gráfica para administrar o banco.

### Verificando

Abra o pgAdmin, conecte com `postgres` + sua senha. Crie um banco chamado `movies`:

- Clique direito em **Databases > Create > Database…**
- Nome: `movies`
- Save.

---

## 13.4 Pacotes NuGet do EF Core

Em um projeto Web API, instale:

```bash
dotnet add package Microsoft.EntityFrameworkCore --version 9.0.8
dotnet add package Microsoft.EntityFrameworkCore.Design --version 9.0.8
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 9.0.4
```

| Pacote | Função |
|---|---|
| `Microsoft.EntityFrameworkCore` | O ORM em si. |
| `Microsoft.EntityFrameworkCore.Design` | Habilita o comando `dotnet ef` (migrations). |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | "**Provider**" do PostgreSQL — ensina o EF a falar com o Postgres. |

> Para SQL Server seria `Microsoft.EntityFrameworkCore.SqlServer`. Para SQLite, `Microsoft.EntityFrameworkCore.Sqlite`.

### Ferramenta global: `dotnet ef`

Além dos pacotes NuGet acima, você precisa instalar **uma vez por máquina** a ferramenta de linha de comando do EF Core:

```bash
dotnet tool install --global dotnet-ef
```

> ⚠️ Sem isso, ao tentar rodar `dotnet ef migrations add ...` você receberá o erro:
> `Could not execute because the specified command or file was not found.`
>
> Se o comando não for reconhecido após a instalação, **feche e reabra o terminal** para o PATH ser atualizado.

---

## 13.5 String de Conexão

A **string de conexão** diz ao EF como achar o banco. Vai no `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5433;Database=movies;Username=postgres;Password=123456"
  }
}
```

| Parte | Significado |
|---|---|
| `Host=localhost` | O servidor está na sua máquina. |
| `Port=5433` | Porta TCP do PostgreSQL. |
| `Database=movies` | Nome do banco. |
| `Username` / `Password` | Credenciais. |

> ⚠️ **Em produção**, NUNCA deixe senhas no `appsettings.json` versionado. Use **User Secrets** ou variáveis de ambiente.

---

## 13.6 DbContext — o coração do EF Core

O **`DbContext`** é a classe que **representa a conexão com o banco**. Ela tem um `DbSet<T>` para cada entidade (tabela).

```csharp
using Microsoft.EntityFrameworkCore;

public class DataContext : DbContext
{
    public DbSet<Movie> Movies { get; set; }
    public DbSet<User> Users { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        var configuration = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json")
            .Build();

        var connectionString = configuration.GetConnectionString("DefaultConnection");
        options.UseNpgsql(connectionString);
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfiguration(new MovieMap());
        modelBuilder.ApplyConfiguration(new UserMap());
    }
}
```

| Membro | Função |
|---|---|
| `DbSet<Movie> Movies` | Representa a tabela `movies`. |
| `OnConfiguring` | Diz **com qual banco** se conectar (string de conexão + provider). |
| `OnModelCreating` | Aplica os **mappings** (configurações de cada tabela). |

> Existem **duas formas** de configurar o `DbContext`:
> 1. **Manualmente** dentro do `OnConfiguring` (como acima) — usado no projeto Movies.API.
> 2. **Por injeção de dependência** com `builder.Services.AddDbContext<...>(...)` no `Program.cs` — mais comum em projetos profissionais.
>
> Vamos usar a **forma 1** no Movies.API por simplicidade didática.

---

## 13.7 Mapeando entidades — pasta `Mappings/`

Cada classe Model corresponde a uma tabela. Para configurar **nomes de colunas, tamanhos, chaves**, criamos uma classe de mapeamento que implementa `IEntityTypeConfiguration<T>`:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

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

### Por que separar em uma classe `MovieMap`?

- **Organização**: cada entidade tem sua configuração própria.
- **Reusabilidade**: fácil de manter.
- **Não polui** o Model com atributos do EF.

---

## 13.8 Migrations — versionando o banco de dados

> **Migration = Git para o seu banco.** Cada migration é um "commit" que descreve uma mudança no schema. Você pode aplicar, voltar atrás, ver o histórico, gerar SQL, comparar versões.

Esta seção é longa de propósito — migrations é **o** tópico onde alunos mais se enrolam.

---

### 13.8.1 O problema que migrations resolve

Imagine um projeto **sem** migrations:

1. Maria cria a tabela `users` direto no pgAdmin com `CREATE TABLE...`.
2. João, no mesmo projeto, na máquina dele, **esquece** de criar.
3. A API roda na Maria e quebra no João.
4. Em produção, ninguém sabe quais colunas existem.
5. Quando alguém renomeia uma coluna, o resto do time fica horas tentando descobrir o porquê.

**Com migrations:**

- A estrutura do banco vive **dentro do código**, no Git.
- Qualquer pessoa clona o projeto, roda `dotnet ef database update` e tem **exatamente o mesmo banco** que todo mundo.
- Cada mudança fica registrada com data, autor e descrição.
- É possível **reverter** uma mudança ruim.
- Em produção, a equipe sabe **exatamente** o que vai mudar antes de aplicar.

---

### 13.8.2 Como migrations funcionam por baixo dos panos

Quando você executa `dotnet ef migrations add Algo`, o EF Core faz isto:

1. **Lê** todas as suas Models e Mappings.
2. **Constrói um modelo** em memória (o "modelo desejado").
3. **Compara** com o snapshot anterior (`DataContextModelSnapshot.cs`).
4. **Gera o diff** — só as mudanças.
5. **Cria um arquivo** `.cs` com dois métodos: `Up()` (aplica) e `Down()` (reverte).
6. **Atualiza o snapshot** para refletir o estado novo.

Quando você executa `dotnet ef database update`:

1. EF olha a tabela especial `__EFMigrationsHistory` no banco.
2. Vê quais migrations **já foram aplicadas**.
3. Executa o `Up()` de cada migration **pendente**, em ordem.
4. Insere uma linha em `__EFMigrationsHistory` para cada uma.

> 🔑 **A tabela `__EFMigrationsHistory`** é o "log de quais migrations já rodaram neste banco". Nunca apague essa tabela — você perderia o histórico e o EF tentaria rodar tudo de novo.

---

### 13.8.3 Pré-requisito: instalar a CLI do EF Core

A CLI (`dotnet ef`) **não vem por padrão** com o .NET SDK. Instale uma vez por máquina:

```bash
dotnet tool install --global dotnet-ef
```

Para atualizar:

```bash
dotnet tool update --global dotnet-ef
```

Para conferir se instalou:

```bash
dotnet ef --version
```

> 💡 Se o comando "não for reconhecido" depois de instalar, **feche e reabra o terminal** (PATH precisa ser recarregado).

---

### 13.8.4 Criando a primeira migration

Na pasta do projeto (onde está o `.csproj`):

```bash
dotnet ef migrations add DatabaseCreation
```

**Resultado** — uma pasta `Migrations/` aparece no projeto, com três arquivos:

```
Migrations/
├── 20260518143022_DatabaseCreation.cs                  ← código da migration
├── 20260518143022_DatabaseCreation.Designer.cs         ← metadados (não mexa)
└── DataContextModelSnapshot.cs                          ← snapshot atual do modelo
```

> 📅 O **timestamp** (`20260518143022`) é o que garante a ordem de aplicação — migrations rodam em ordem cronológica de criação.

#### Anatomia do arquivo `_DatabaseCreation.cs`

```csharp
public partial class DatabaseCreation : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "movies",
            columns: table => new
            {
                id = table.Column<int>(type: "integer", nullable: false)
                    .Annotation("Npgsql:ValueGenerationStrategy",
                        NpgsqlValueGenerationStrategy.IdentityByDefaultColumn),
                title = table.Column<string>(type: "character varying(100)",
                    maxLength: 100, nullable: false),
                // ... demais colunas
            },
            constraints: table => table.PrimaryKey("PK_movies", x => x.id));
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "movies");
    }
}
```

| Método | O que faz |
|---|---|
| `Up()` | **Aplica** a mudança (criar tabela, adicionar coluna, etc). Roda no `database update`. |
| `Down()` | **Reverte** a mudança. Roda quando você volta para uma migration anterior. |

---

### 13.8.5 Aplicando no banco

```bash
dotnet ef database update
```

O que acontece:

1. EF se conecta usando a connection string do `DataContext`.
2. Cria a tabela `__EFMigrationsHistory` (se não existir).
3. Roda o `Up()` de toda migration ainda **não aplicada**.
4. Registra cada uma em `__EFMigrationsHistory`.

**Confira no pgAdmin:** as tabelas (`movies`, `users`) devem aparecer no banco, junto com `__EFMigrationsHistory`.

#### Aplicar até uma migration específica

```bash
dotnet ef database update DatabaseCreation
```

Útil para **reverter**: aplica/desfaz tudo até chegar **naquela** migration.

#### Reverter TUDO (banco volta ao zero)

```bash
dotnet ef database update 0
```

> ⚠️ Isso roda `Down()` de **todas** as migrations. Não use em produção sem ter certeza absoluta.

---

### 13.8.6 Ciclo de vida típico de uma mudança no banco

Você quer **adicionar a coluna `release_year` em `movies`**. Fluxo correto:

#### 1. Alterar a Model

```csharp
public class Movie
{
    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public string PosterUrl { get; set; } = string.Empty;
    public string Overview { get; set; } = string.Empty;
    public int ReleaseYear { get; set; }   // ← NOVA propriedade
}
```

#### 2. Alterar o Mapping

```csharp
builder.Property(x => x.ReleaseYear)
       .HasColumnName("release_year")
       .IsRequired();
```

#### 3. Criar a migration

```bash
dotnet ef migrations add AddReleaseYearToMovies
```

EF compara, detecta a nova coluna, e gera:

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<int>(
        name: "release_year",
        table: "movies",
        type: "integer",
        nullable: false,
        defaultValue: 0);
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropColumn(name: "release_year", table: "movies");
}
```

#### 4. Revisar o arquivo gerado

**Sempre abra e leia.** Confira se o `Up()` faz o que você esperava. Se a tabela já tem dados e a coluna é `NOT NULL`, o `defaultValue: 0` salva — caso contrário, ajuste manualmente.

#### 5. Aplicar

```bash
dotnet ef database update
```

#### 6. Commit no Git

Os três arquivos da pasta `Migrations/` (incluindo o `DataContextModelSnapshot.cs`) **vão para o Git**. Migrations são parte do código do projeto.

---

### 13.8.7 Nomeando migrations corretamente

Bons nomes contam uma **história** ao ler o histórico:

| ✅ Bom | ❌ Ruim |
|---|---|
| `AddReleaseYearToMovies` | `Update1` |
| `RenameUserEmailColumn` | `fixstuff` |
| `CreateOrdersTable` | `nova` |
| `MakeMovieOverviewOptional` | `migration2` |

**Padrão:** verbo + alvo. Em inglês, geralmente — é o padrão da comunidade .NET.

---

### 13.8.8 Comandos essenciais (cola rápida)

```bash
# Listar migrations existentes (aplicadas e pendentes)
dotnet ef migrations list

# Criar nova migration
dotnet ef migrations add NomeDescritivo

# Remover a ÚLTIMA migration (somente se NÃO foi aplicada ao banco)
dotnet ef migrations remove

# Aplicar todas as migrations pendentes
dotnet ef database update

# Aplicar até uma migration específica (pra frente ou pra trás)
dotnet ef database update NomeDaMigration

# Reverter TODAS (banco zerado)
dotnet ef database update 0

# Gerar SQL bruto (sem executar — útil para produção)
dotnet ef migrations script

# Gerar SQL entre duas migrations específicas
dotnet ef migrations script DePrimeira AteSegunda

# Apagar o banco inteiro (drop database)
dotnet ef database drop
```

> ⚠️ `database drop` **apaga TUDO** — inclusive os dados. Use apenas em dev.

---

### 13.8.9 Pelo Visual Studio (Package Manager Console)

Se preferir não usar terminal, abra **Tools → NuGet Package Manager → Package Manager Console** e use os equivalentes:

| CLI | Package Manager Console |
|---|---|
| `dotnet ef migrations add Nome` | `Add-Migration Nome` |
| `dotnet ef migrations remove` | `Remove-Migration` |
| `dotnet ef database update` | `Update-Database` |
| `dotnet ef database update Nome` | `Update-Database Nome` |
| `dotnet ef migrations list` | `Get-Migration` |
| `dotnet ef migrations script` | `Script-Migration` |
| `dotnet ef database drop` | `Drop-Database` |

> 💡 O Package Manager Console executa no contexto do **projeto selecionado** no dropdown "Default project". Se tiver mais de um projeto na solução, confira esse dropdown.

---

### 13.8.10 Problemas comuns e como resolver

#### "No project was found"
Você está rodando o comando da pasta errada. Vá até a pasta do `.csproj` antes de executar.

#### "Build failed"
O projeto precisa **compilar** para o EF ler as Models. Rode `dotnet build` antes e corrija erros.

#### "No DbContext was found"
Falta o pacote `Microsoft.EntityFrameworkCore.Design`, OU o `DataContext` não está acessível (verifique se a classe é `public` e o namespace está certo).

#### "The migration has already been applied to the database"
Você tentou `migrations remove` numa migration que já foi aplicada. **Primeiro** reverta no banco:
```bash
dotnet ef database update NomeDaMigrationAnterior
dotnet ef migrations remove
```

#### "Pending model changes"
Você alterou Models/Mappings e esqueceu de gerar a migration. Rode `dotnet ef migrations add Algo`.

#### Conflito de migrations em equipe
Dois devs criaram migrations em paralelo no mesmo branch. Solução:
1. Quem rebase **deleta sua migration**, faz pull.
2. Cria a migration novamente — agora ela vem depois da do colega.
3. Roda `database update` para sincronizar.

#### Esqueci de fazer migration de uma mudança grande
Não tem volta: a migration atual vai conter **tudo** que mudou desde a última. Por isso a regra: **uma migration por mudança lógica**. Não acumule.

---

### 13.8.11 Migrations em produção

Em ambiente real, você **não** roda `dotnet ef database update` direto no servidor. Padrões profissionais:

1. **Gerar script SQL** localmente e revisar:
   ```bash
   dotnet ef migrations script --idempotent -o migration.sql
   ```
   - `--idempotent` gera um SQL que **verifica** o que já está aplicado e só roda o que falta. Pode rodar quantas vezes quiser sem efeito colateral.

2. **DBA revisa o SQL** antes de executar (em times grandes).

3. **Executar via pipeline CI/CD**: GitHub Actions, Azure DevOps, etc. aplicam o script no banco como parte do deploy.

4. **Backup do banco ANTES de qualquer migration em produção** — sempre.

5. **Migrações destrutivas (DROP COLUMN, RENAME)** exigem cuidado especial: faça em **duas etapas** (release N adiciona, release N+1 remove) para permitir rollback.

---

### 13.8.12 Regras de ouro

1. ✅ **Sempre nomeie** migrations descritivamente.
2. ✅ **Sempre revise** o arquivo gerado antes de aplicar.
3. ✅ **Commite** a pasta `Migrations/` no Git.
4. ✅ **Backup antes** de migrar em produção.
5. ❌ **Nunca edite** uma migration **já aplicada em produção**. Crie uma nova migration corretiva.
6. ❌ **Nunca apague** a tabela `__EFMigrationsHistory`.
7. ❌ **Nunca altere** o banco manualmente (pgAdmin, SQL direto) em coisas que o EF gerencia — gere uma migration.

---



## 13.9 CRUD com EF Core — exemplos práticos

Assumindo que `connection` é uma instância de `DataContext`:

### Criar (INSERT)

```csharp
var movie = new Movie("Matrix", "http://...", "Filme sobre...");
connection.Movies.Add(movie);
connection.SaveChanges();   // ← só aqui o SQL é executado
```

### Ler (SELECT)

```csharp
// Listar todos
var todos = connection.Movies.ToList();

// Buscar um
var um = connection.Movies.FirstOrDefault(m => m.Id == 1);

// Filtrar
var matrixes = connection.Movies.Where(m => m.Title.Contains("Matrix")).ToList();
```

### Atualizar (UPDATE)

```csharp
var movie = connection.Movies.FirstOrDefault(m => m.Id == 1);
if (movie != null)
{
    movie.Title = "Matrix Reloaded";
    connection.SaveChanges();
}
```

### Excluir (DELETE)

```csharp
var movie = connection.Movies.FirstOrDefault(m => m.Id == 1);
if (movie != null)
{
    connection.Movies.Remove(movie);
    connection.SaveChanges();
}
```

---

## 13.10 `AsNoTracking()` — leitura mais rápida

Por padrão, o EF "**rastreia**" todas as entidades que você consulta — para detectar mudanças quando você chamar `SaveChanges()`. Isso consome memória e CPU.

Quando você só vai **ler** (sem editar), use `AsNoTracking()`:

```csharp
var todos = connection.Movies.AsNoTracking().ToList();   // mais rápido!
var um = connection.Movies.AsNoTracking().FirstOrDefault(m => m.Id == 1);
```

> **Regra prática**: `AsNoTracking()` em **toda consulta de leitura** que vai ser devolvida pela API. Você não vai editar mesmo.

---

## 13.11 Padrão usado no Movies.API

No projeto **Movies.API** (que vamos construir no Cap. 15), os Services criam o `DataContext` manualmente:

```csharp
public class MovieService : IRepositoryMovie
{
    public Movie? GetById(int id)
    {
        using var connection = new DataContext();   // cria
        try
        {
            return connection.Movies
                             .AsNoTracking()
                             .FirstOrDefault(x => x.Id == id);
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return null;
        }
        // o `using` descarta a conexão automaticamente no fim
    }
}
```

Por que `using var connection = new DataContext();`?

- O `using` garante que a conexão será **fechada** ao final do método (chama `Dispose()`).
- Cada operação tem **sua própria conexão** — independência total.
- É o estilo **didático/simples**. Em projetos grandes, usa-se DI com `AddDbContext`.

---

## 13.12 Ordem de execução EF Core (resumo)

```
1. Você cria/altera Models (classes C#)
2. Cria/atualiza Mappings (IEntityTypeConfiguration)
3. Configura DbContext (DbSet, OnConfiguring, OnModelCreating)
4. Roda: dotnet ef migrations add <Nome>
5. Roda: dotnet ef database update
6. Tabelas existem no banco. Aplicação pode usá-las.
```

---

## 13.13 Resumo do capítulo

- **Banco relacional** = tabelas. **PostgreSQL** é gratuito e robusto.
- **EF Core** é o ORM oficial do .NET. Traduz objetos em SQL.
- **DbContext** representa a conexão; **DbSet<T>** representa cada tabela.
- **Mappings** (`IEntityTypeConfiguration<T>`) configuram cada entidade.
- **Migrations** versionam o schema do banco.
- Use **`AsNoTracking()`** em leituras.

---

## 13.14 Exercícios

1. Instale o PostgreSQL e crie um banco chamado `escola`.
2. Em um novo projeto, crie a classe `Aluno` (`Id`, `Nome`, `Idade`).
3. Crie um `AlunoMap` mapeando para a tabela `alunos`.
4. Crie um `DataContext` com `DbSet<Aluno>` e configure a connection string.
5. Crie a primeira migration e aplique no banco. Confira no pgAdmin se a tabela apareceu.
6. Crie um pequeno Console que insere 3 alunos e depois lista todos.

---

➡️ **Próximo capítulo:** [Capítulo 14 — Autenticação com JWT](14-Autenticacao-JWT.md)
