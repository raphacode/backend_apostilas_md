# Capítulo 9 — Fundamentos de API e HTTP

> Antes de criar uma API, você precisa entender como **a internet conversa**. Este capítulo explica os tijolos que sustentam toda a web.

---

## 9.1 O que é uma API (revisão e aprofundamento)

No Capítulo 1 vimos que uma API é uma "porta de entrada" para sistemas conversarem. Agora vamos aprofundar.

### Definição mais técnica

> **API (Application Programming Interface)**: um conjunto de **regras e endereços padronizados** que permite que dois sistemas troquem informações de forma previsível.

Uma API expõe um conjunto de **endpoints** (URLs específicas), cada um aceitando determinados **métodos** e **dados**, e devolvendo determinadas **respostas**.

### Tipos de API

- **API REST** (a mais comum hoje, e o foco desta apostila).
- **API SOAP** (mais antiga, ainda usada em sistemas corporativos legados).
- **GraphQL** (alternativa moderna, mais flexível).
- **gRPC** (de altíssima performance, comum entre microsserviços).

Vamos focar em **REST**.

---

## 9.2 O que é REST?

**REST** = **REpresentational State Transfer**.

É um **estilo arquitetural** (não é um protocolo) baseado em alguns princípios:

1. **Cliente-servidor**: separação clara entre quem consome e quem provê.
2. **Stateless**: cada requisição é **independente**. O servidor não guarda o "estado" do cliente entre requisições.
3. **Recursos**: tudo é tratado como um **recurso** identificado por uma URL (ex: `/usuarios/42`).
4. **Verbos HTTP**: as ações são feitas pelos métodos HTTP padrão (`GET`, `POST`, `PUT`, `DELETE`).
5. **Representações**: os recursos trafegam como **JSON** (geralmente), mas poderia ser XML, HTML, etc.

### Em uma frase

> Uma **API REST** é uma API onde cada coisa importante tem uma **URL**, e a gente **age sobre ela** usando os verbos do HTTP.

---

## 9.3 HTTP — o protocolo da web

**HTTP** = **HyperText Transfer Protocol**. É a "linguagem" usada pelos navegadores e APIs para conversar pela internet.

Toda comunicação HTTP é um **par**: **requisição** (do cliente) → **resposta** (do servidor).

### Anatomia de uma requisição HTTP

```
GET /usuarios/42 HTTP/1.1
Host: api.exemplo.com
Authorization: Bearer abc123
Content-Type: application/json

{
  "campoOpcional": "valor"
}
```

Componentes:

| Parte | Significado |
|---|---|
| `GET` | **Verbo HTTP** (método). |
| `/usuarios/42` | **Caminho/rota** (URL relativa). |
| `HTTP/1.1` | Versão do protocolo. |
| `Host: api.exemplo.com` | **Header**: domínio do servidor. |
| `Authorization: ...` | **Header** com credenciais. |
| `Content-Type: ...` | **Header**: tipo do corpo. |
| O JSON do final | **Body** (corpo) — opcional. |

### Anatomia de uma resposta HTTP

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 87

{
  "id": 42,
  "nome": "Maria",
  "email": "maria@exemplo.com"
}
```

Componentes:

| Parte | Significado |
|---|---|
| `200 OK` | **Status code** (e descrição). |
| `Content-Type` | Tipo do corpo retornado. |
| `Content-Length` | Tamanho em bytes. |
| O JSON | **Body** com a resposta. |

---

## 9.4 Verbos HTTP

Os verbos indicam **a intenção da requisição**.

| Verbo | Uso típico | "Significado" |
|---|---|---|
| `GET` | **Buscar/consultar** dados | "Me dá X" |
| `POST` | **Criar** algo novo | "Cria isso aqui" |
| `PUT` | **Atualizar** algo (substituição completa) | "Substitui X por isso" |
| `PATCH` | **Atualizar parcialmente** | "Muda só o campo Y" |
| `DELETE` | **Excluir** | "Apaga X" |
| `OPTIONS` | Consultar o que é permitido | "O que posso fazer aqui?" |

### Exemplos práticos

| Requisição | Significado |
|---|---|
| `GET /produtos` | Listar todos os produtos |
| `GET /produtos/15` | Pegar o produto de id 15 |
| `POST /produtos` | Cadastrar um novo produto (dados no body) |
| `PUT /produtos/15` | Atualizar o produto 15 (substitui completo) |
| `PATCH /produtos/15` | Atualizar só alguns campos do produto 15 |
| `DELETE /produtos/15` | Apagar o produto 15 |

### Idempotência (conceito importante)

Um verbo é **idempotente** se chamá-lo **várias vezes** dá o **mesmo resultado** que chamá-lo uma vez.

| Verbo | Idempotente? |
|---|---|
| `GET` | ✅ Sim (consultar é sempre o mesmo) |
| `PUT` | ✅ Sim (atualizar para o mesmo estado) |
| `DELETE` | ✅ Sim (apagar de novo o que já foi apagado dá o mesmo) |
| `POST` | ❌ Não (cada chamada cria um novo registro) |
| `PATCH` | Geralmente não |

Isso importa: clientes podem **repetir** requisições idempotentes em caso de erro, sem medo de duplicar dados.

---

## 9.5 Status Codes

São números de 3 dígitos que dizem **o que aconteceu** com a requisição. Agrupados em categorias:

| Faixa | Categoria | Significado |
|---|---|---|
| **1xx** | Informacional | Raramente usado |
| **2xx** | **Sucesso** | Deu certo |
| **3xx** | Redirecionamento | "Vai pra outro lugar" |
| **4xx** | **Erro do cliente** | Você errou no pedido |
| **5xx** | **Erro do servidor** | Eu (servidor) bugei |

### Os mais importantes

| Code | Nome | Quando usar |
|---|---|---|
| **200** | OK | Sucesso geral |
| **201** | Created | Algo foi **criado** com sucesso (POST) |
| **204** | No Content | Sucesso, sem corpo de resposta (DELETE típico) |
| **400** | Bad Request | A requisição está malformada / dados inválidos |
| **401** | Unauthorized | Não autenticado (precisa logar) |
| **403** | Forbidden | Autenticado, mas **sem permissão** |
| **404** | Not Found | Recurso não existe |
| **409** | Conflict | Conflito (ex: e-mail já cadastrado) |
| **422** | Unprocessable Entity | Validação falhou |
| **500** | Internal Server Error | Erro inesperado no servidor |
| **503** | Service Unavailable | Servidor fora do ar |

> **Diferença sutil entre 401 e 403:**
> - **401**: você não disse quem é.
> - **403**: eu sei quem você é, mas você não pode fazer isso.

---

## 9.6 JSON

**JSON** = **JavaScript Object Notation**. É o **formato padrão** de troca de dados em APIs modernas.

### Por que JSON?

- **Legível por humanos**.
- **Simples** de gerar e processar.
- **Suportado por todas as linguagens**.

### Sintaxe

```json
{
  "id": 42,
  "nome": "Maria",
  "ativo": true,
  "salario": 3500.50,
  "tags": ["admin", "premium"],
  "endereco": {
    "rua": "Av. Brasil",
    "numero": 123
  },
  "filhos": null
}
```

### Tipos suportados

| Tipo | Exemplo |
|---|---|
| string | `"Maria"` |
| número | `42`, `3.14` |
| boolean | `true`, `false` |
| null | `null` |
| objeto | `{ }` |
| array | `[ ]` |

### Regras importantes

- **Chaves entre aspas duplas**: `"nome"`, **nunca** `'nome'` ou sem aspas.
- **Vírgula separa** campos, **mas não no último**.
- **Strings sempre com aspas duplas**.

### JSON em C#

C# tem suporte nativo via `System.Text.Json`:

```csharp
using System.Text.Json;

var pessoa = new { Nome = "Maria", Idade = 25 };
string json = JsonSerializer.Serialize(pessoa);
// {"Nome":"Maria","Idade":25}

string entrada = "{\"Nome\":\"João\",\"Idade\":30}";
var p = JsonSerializer.Deserialize<Pessoa>(entrada);
```

ASP.NET Core faz isso **automaticamente** para você na maioria dos casos.

---

## 9.7 Endpoints e Rotas

Um **endpoint** é uma "porta" da API: um par **verbo + caminho**. Exemplos:

```
GET    /produtos
GET    /produtos/{id}
POST   /produtos
PUT    /produtos/{id}
DELETE /produtos/{id}
```

**Rota** é o **caminho** propriamente dito (`/produtos/{id}`). Um endpoint é a rota **+** o método.

### Padrões de rotas REST

Para um recurso `produtos`:

| Endpoint | Função |
|---|---|
| `GET /produtos` | Lista todos |
| `GET /produtos/15` | Busca um |
| `POST /produtos` | Cria |
| `PUT /produtos/15` | Atualiza |
| `DELETE /produtos/15` | Remove |

### Recursos aninhados

Quando há relação entre recursos:

```
GET  /usuarios/42/pedidos          ← pedidos do usuário 42
POST /usuarios/42/pedidos          ← novo pedido para o usuário 42
GET  /usuarios/42/pedidos/7        ← pedido 7 do usuário 42
```

---

## 9.8 Parâmetros — Route, Query, Body, Header

Existem **quatro lugares** onde uma requisição pode carregar dados.

### 1. Parâmetros de Rota (Route)

Vão **dentro do caminho** da URL:

```
GET /produtos/15
GET /usuarios/42/pedidos/7
```

Servem para **identificar um recurso específico**. São **obrigatórios** na rota.

### 2. Query String

Vêm **depois do `?`** na URL, no formato `chave=valor`:

```
GET /produtos?categoria=eletronicos&precoMax=1000&ordenar=preco
```

Servem para **filtrar, paginar, ordenar**. São **opcionais** geralmente.

### 3. Body (corpo)

São **dados enviados no corpo** da requisição (geralmente JSON). Usado em **`POST`**, **`PUT`**, **`PATCH`**.

```http
POST /produtos
Content-Type: application/json

{
  "nome": "Notebook",
  "preco": 4500.00
}
```

### 4. Headers

Informações **complementares** da requisição:

```
Authorization: Bearer eyJhbG...
Accept-Language: pt-BR
Content-Type: application/json
```

Usados para autenticação, idioma, formato, etc.

### Quando usar cada um?

| Situação | Use |
|---|---|
| Identificar recurso único | **Route** |
| Filtrar/buscar/paginar | **Query** |
| Enviar dados grandes / criar / atualizar | **Body** |
| Autenticação / metadados | **Header** |

---

## 9.9 Como testar uma API?

Antes de criar uma, é bom saber **testar**. Ferramentas:

- **Postman** (mais famosa, GUI).
- **Insomnia** (similar, mais leve).
- **Thunder Client** (extensão do VS Code).
- **curl** (linha de comando).
- **OpenAPI nativo do ASP.NET Core** para gerar o documento da API.
- **Swagger UI** como interface interativa de testes, instalada por pacote quando você quiser uma tela navegável.

Exemplo com `curl`:

```bash
curl -X GET https://api.exemplo.com/produtos
curl -X POST https://api.exemplo.com/produtos \
  -H "Content-Type: application/json" \
  -d '{"nome":"Notebook","preco":4500}'
```

---

## 9.10 Boas práticas em APIs REST

1. **Use substantivos no plural** para recursos: `/produtos`, não `/produto`, nem `/listarProdutos`.
2. **Use os verbos HTTP corretamente.** Não faça `GET /deletarProduto`.
3. **Status codes adequados.** Retornar `200 OK` quando deu erro é pecado mortal.
4. **Versione sua API**: `/api/v1/produtos`. Quando quiser mudar, crie `/api/v2/`.
5. **Padronize as respostas**: sempre o mesmo formato de erro, por exemplo.
6. **Documente**: use **OpenAPI** para descrever os endpoints e **Swagger UI** para testar pelo navegador.
7. **Valide entradas**: nunca confie no que o cliente manda.
8. **Pagine listas grandes**: nunca devolva 1 milhão de itens de uma vez.
9. **HTTPS sempre**, em produção. Nunca HTTP puro.
10. **Não exponha dados sensíveis** (senha, hash, dados internos do banco) em respostas.

---

## 9.11 Resumo do capítulo

- API REST é o estilo mais usado para APIs hoje.
- HTTP é o protocolo: cada interação é **requisição → resposta**.
- **Verbos** indicam intenção: `GET`, `POST`, `PUT`, `DELETE`...
- **Status codes** comunicam o resultado (200, 201, 400, 404, 500...).
- **JSON** é o formato padrão de troca de dados.
- **Endpoints** = verbo + rota. Existem 4 tipos de parâmetros: **route, query, body, header**.

---

## 9.12 Exercícios (sem código)

1. Para cada situação, indique o **verbo HTTP** e a **rota**:
   - Listar todos os clientes.
   - Pegar o cliente de ID 7.
   - Cadastrar um novo cliente.
   - Atualizar dados do cliente 7.
   - Excluir o cliente 7.
   - Listar os pedidos do cliente 7.
2. Que **status code** retornar?
   - Cadastro feito com sucesso.
   - Cliente enviou e-mail mal formatado.
   - Recurso não existe.
   - Servidor caiu.
3. Quando usar **query** vs **route** vs **body**? Dê 1 exemplo de cada.
4. Escreva um JSON representando uma "tarefa" com `id`, `titulo`, `concluida`, `prazo`.

---

➡️ **Próximo capítulo:** [Capítulo 10 — Construindo uma API REST com ASP.NET Core](10-Construindo-API-REST-ASPNET-Core.md)
