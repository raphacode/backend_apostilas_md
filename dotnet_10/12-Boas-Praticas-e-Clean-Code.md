# Capítulo 12 — Boas Práticas e Clean Code

> Programar é fácil. Programar **bem** é o que diferencia um profissional. Este capítulo traz princípios que vão te acompanhar a carreira inteira.

---

## 12.1 O que é Clean Code?

**Clean Code** é um conjunto de práticas para escrever **código legível, simples e manutenível**. Vem do livro homônimo de Robert C. Martin ("Uncle Bob").

A ideia central:

> **"Código é lido muito mais vezes do que é escrito."**

Você escreve uma função uma vez. Mas você (ou outros) vão ler aquela função **dezenas de vezes** depois — para entender, para mudar, para corrigir bugs.

**Optimize para o leitor, não para o escritor.**

---

## 12.2 Nomes — a regra mais importante

Bons nomes são **70% da legibilidade do código**.

### Nomes descritivos

❌ Ruim:
```csharp
int d;          // o que é "d"?
List<int> l;    // l de quê?
void Func1() {} // ???
```

✅ Bom:
```csharp
int diasDesdeUltimoLogin;
List<int> idsDeUsuariosAtivos;
void EnviarEmailDeBoasVindas() {}
```

### Convenções em C#

| Tipo | Convenção | Exemplo |
|---|---|---|
| Classes / Interfaces | **PascalCase** | `Usuario`, `IRepositorio` |
| Métodos / Propriedades | **PascalCase** | `CalcularTotal()` |
| Variáveis locais / parâmetros | **camelCase** | `precoTotal` |
| Campos privados | **`_camelCase`** | `_logger`, `_repositorio` |
| Constantes | **PascalCase** ou MAIÚSCULAS | `MaxTentativas` |
| Interfaces | Prefixo **`I`** | `IUsuarioService` |

### Verbo para método, substantivo para classe/atributo

```csharp
public class Usuario { }                     // substantivo
public bool EhMaiorDeIdade() { return ... }  // verbo
public string Nome { get; set; }             // substantivo
```

### Evite abreviações enigmáticas

❌ `usrSvc`, `prdMgr`, `tmp`, `aux`
✅ `usuarioService`, `gerenciadorDeProdutos`

---

## 12.3 Funções (métodos) curtas

> "**Funções devem ser pequenas. E menores que isso.**" — Uncle Bob

**Regra prática:** se uma função não cabe na sua tela sem rolagem, ela é grande demais.

### Faça uma coisa só

Se você usa "**E**" para descrever o que a função faz, ela está fazendo coisas demais:

❌ `ValidarEEnviarEmail()` → duas funções: `Validar` e `EnviarEmail`.

### Poucos parâmetros

- 0 ou 1 parâmetro: ótimo.
- 2: aceitável.
- 3: pense duas vezes.
- 4 ou mais: provavelmente você precisa de uma classe agrupando esses dados.

❌
```csharp
void CriarPedido(int idCliente, string endereco, string cidade,
    string estado, string cep, double valor, double frete, ...) {}
```

✅
```csharp
void CriarPedido(Pedido pedido) {}
```

### Nível de abstração consistente

Dentro de uma função, fique no **mesmo nível** de detalhe:

❌
```csharp
public void ProcessarPedido(Pedido p)
{
    ValidarPedido(p);
    var connection = new SqlConnection("...");  // detalhe técnico misturado!
    connection.Open();
    var cmd = connection.CreateCommand();
    // ...
    EnviarEmailDeConfirmacao(p);
}
```

✅
```csharp
public void ProcessarPedido(Pedido p)
{
    ValidarPedido(p);
    SalvarNoBanco(p);
    EnviarEmailDeConfirmacao(p);
}
```

---

## 12.4 SRP — Single Responsibility Principle

> **Uma classe deve ter um, e apenas um, motivo para mudar.**

Cada classe tem **uma responsabilidade**. Não misture.

### Anti-exemplo

```csharp
public class Usuario
{
    public string Nome { get; set; }

    public void SalvarNoBanco() { /* SQL aqui */ }
    public void EnviarEmail() { /* SMTP aqui */ }
    public string FormatarHtml() { /* HTML aqui */ }
}
```

Aqui `Usuario` tem **três motivos** para mudar: regras de negócio, banco e e-mail. Isso é SRP violado.

### Refatorado

```csharp
public class Usuario { /* só dados */ }

public class UsuarioRepository { void Salvar(Usuario u); }
public class UsuarioMailer { void EnviarBoasVindas(Usuario u); }
public class UsuarioFormatter { string ParaHtml(Usuario u); }
```

---

## 12.5 DRY — Don't Repeat Yourself

> **Cada pedaço de conhecimento deve existir em um único lugar.**

Se você está **copiando e colando código**, pare. Crie um método/classe.

### Anti-exemplo

```csharp
double precoFinalProduto1 = preco1 * 1.1 + 5;
double precoFinalProduto2 = preco2 * 1.1 + 5;
double precoFinalProduto3 = preco3 * 1.1 + 5;
```

### Refatorado

```csharp
double CalcularPrecoFinal(double preco) => preco * 1.1 + 5;

double precoFinalProduto1 = CalcularPrecoFinal(preco1);
// ...
```

Agora se a fórmula mudar, você muda **um lugar**.

> ⚠️ **Cuidado com DRY excessivo.** Duas coisas que **parecem iguais** mas evoluem por motivos diferentes **não** devem ser unificadas. Tem um princípio chamado **AHA — Avoid Hasty Abstractions**.

---

## 12.6 KISS — Keep It Simple, Stupid

> **A solução mais simples que funciona é a melhor.**

Não invente complicações. Iniciantes muitas vezes querem mostrar serviço com padrões mirabolantes — e criam um **monstro**.

### Princípio

- Resista a abstrações prematuras.
- Não crie interface se só haverá uma implementação no futuro previsível.
- Não adicione opções "para o caso de…". **Adicione quando precisar.**

> "Faça funcionar. Faça certo. Faça rápido." — Kent Beck. **Nessa ordem.**

---

## 12.7 YAGNI — You Aren't Gonna Need It

> **Não implemente o que você acha que vai precisar. Implemente o que você precisa AGORA.**

Cada linha de código não escrita é uma linha sem bug. Adicione recursos quando o requisito **realmente aparecer**.

---

## 12.8 Comentários: use com sabedoria

❌ Comentário **óbvio**:
```csharp
// Incrementa contador
contador++;
```

❌ Comentário **mentiroso** (código mudou, comentário não):
```csharp
// Calcula o desconto de 10%
return preco * 0.85;
```

✅ Comentário que **explica o porquê**:
```csharp
// Workaround para o bug #4521 do SDK do gateway de pagamento.
// Pode ser removido quando atualizarmos para a versão 5+.
Thread.Sleep(200);
```

> **A melhor documentação é o próprio código bem escrito.** Comente apenas o que o código não consegue dizer.

---

## 12.9 Tratamento de erros

### Use exceções para erros excepcionais

Não use exceção para fluxo normal:

❌
```csharp
try
{
    var u = Buscar(id);
}
catch (UsuarioNaoEncontradoException)
{
    return null;
}
```

✅
```csharp
public Usuario? Buscar(int id) { ... }
// retornar null para "não encontrado" é normal
```

### Não engula exceções silenciosamente

❌
```csharp
try { /* ... */ }
catch { } // SUMIU. Adeus, debug.
```

✅
```csharp
try { /* ... */ }
catch (Exception ex)
{
    _logger.LogError(ex, "Falha ao processar...");
    throw; // ou trate de verdade
}
```

### Crie exceções específicas quando fizer sentido

```csharp
public class SaldoInsuficienteException : Exception
{
    public SaldoInsuficienteException(string msg) : base(msg) { }
}
```

Permite tratar o erro com precisão lá em cima.

---

## 12.10 Separação de responsabilidades em projeto

Já vimos no Capítulo 11. Reforçando:

| Camada | Responsabilidade |
|---|---|
| Controller | Traduzir HTTP ↔ Service |
| Service | Regras de negócio |
| Repository | Acesso a dados |
| DTO | Trafegar dados |
| Model | Representar entidades |

**Não misture.** Controller que faz SQL é cheiro de código ruim.

---

## 12.11 Organização de arquivos e pastas

- **Uma classe por arquivo.** Nome do arquivo = nome da classe.
- **Pastas por responsabilidade** (ou por feature em projetos grandes).
- **Nomes consistentes** — escolha um padrão (português ou inglês) e mantenha.
- **Não use** `Utils`, `Helpers`, `Common`, `Misc` como pasta principal — vira lixeira.

---

## 12.12 Princípios SOLID — visão geral

Cinco princípios que guiam código orientado a objetos:

| Letra | Princípio | Em uma frase |
|---|---|---|
| **S** | **S**ingle Responsibility | Cada classe, **uma responsabilidade**. |
| **O** | **O**pen/Closed | Aberto para **extensão**, fechado para **modificação**. |
| **L** | **L**iskov Substitution | Subclasses devem **substituir** a base sem quebrar. |
| **I** | **I**nterface Segregation | Várias interfaces específicas > uma gigante. |
| **D** | **D**ependency Inversion | Dependa de **abstrações**, não de implementações. |

> Estude SOLID a fundo depois desta apostila. É um divisor de águas.

---

## 12.13 Code review e linting

- **Revise o código de outros** e **peça revisão do seu**. É como aprendemos.
- Use **EditorConfig** para padronizar formatação no projeto.
- O **C# Dev Kit** já avisa muitos problemas no VS Code.
- **Formate sempre** antes de commitar (`Alt + Shift + F`).

---

## 12.14 Por que pensar em manutenção?

Estudos mostram que **70-80% do custo** de um software ao longo da vida é em **manutenção**, não em construção.

- Código **bagunçado** = bugs intermitentes, lentidão para entregar, devs frustrados.
- Código **limpo** = mudanças rápidas, testes possíveis, time feliz.

Quando você escreve uma função feia agora porque "está com pressa", você está **emprestando dinheiro do seu eu do futuro com juros altos**. Isso se chama **dívida técnica**.

---

## 12.15 Checklist de qualidade antes de "fechar" código

Antes de considerar uma feature pronta, pergunte-se:

- [ ] Os **nomes** estão descritivos?
- [ ] As **funções estão pequenas** e fazem uma coisa só?
- [ ] As **classes têm uma responsabilidade**?
- [ ] Não há **código duplicado**?
- [ ] **Casos de erro** estão tratados (mas sem exagero)?
- [ ] **Validação** de entradas está em pé?
- [ ] Os **dados sensíveis** estão protegidos?
- [ ] O **OpenAPI/Swagger UI** documenta corretamente?
- [ ] A **API** segue convenções REST?
- [ ] O **código está formatado** consistentemente?
- [ ] Não há **`Console.WriteLine`** de debug esquecido?
- [ ] Não há **comentários TODO** críticos esquecidos?

---

## 12.16 Próximos passos na sua jornada

Ao terminar essa apostila, você está pronto para mergulhar em:

1. **Entity Framework Core** — para acessar bancos de dados de verdade.
2. **Autenticação e autorização** com JWT / Identity.
3. **Testes automatizados** com xUnit.
4. **Padrões de projeto** (Repository, Unit of Work, CQRS, Mediator…).
5. **SOLID** em profundidade.
6. **Docker** e implantação na nuvem (Azure / AWS).
7. **Mensageria** (RabbitMQ, Kafka).
8. **Microsserviços**.
9. **Arquitetura limpa** (Clean Architecture, Hexagonal).

A apostila te deu **a base sólida**. O resto se constrói **fazendo**.

---

## 12.17 Resumo do capítulo

- Bons **nomes** são tudo.
- **Funções curtas**, fazendo **uma coisa**, com **poucos parâmetros**.
- **SRP, DRY, KISS, YAGNI** — internalize.
- **Comentários** explicam o **porquê**, não o **quê**.
- **Trate exceções** com responsabilidade.
- **Separação por camadas** mantém o projeto saudável.
- O verdadeiro custo do código está na **manutenção**.

---

## 12.18 Exercícios finais

1. Pegue um código antigo seu (ou de exemplo) e **renomeie variáveis** com nomes descritivos.
2. Identifique uma função grande e **divida em funções menores**.
3. Procure **código duplicado** em um projeto seu e refatore.
4. Para cada classe sua, pergunte-se: "**qual a única responsabilidade dela?**". Se não conseguir responder em uma frase, refatore.
5. Adicione **validação adequada** em todos os endpoints da sua API.
6. Implemente um **middleware de log** que registra todas as requisições recebidas.

---

## 🎓 Encerramento

Parabéns! Se você chegou aqui — leu, praticou, fez os exercícios — você **deixou de ser iniciante** em backend com C#.

Lembre-se:

> **Ninguém aprende programação em uma apostila.** Aprende-se **construindo coisas**.

Crie projetos. Erre. Conserte. Pesquise. Pergunte. Releia.

A jornada está só começando — e o caminho é divertido. Boa sorte!

---

⬅️ **Voltar ao [Sumário](00-Index.md)**
