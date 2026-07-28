# Capítulo 9 — Fundamentos de API e HTTP

> Antes de criar uma API, você precisa entender como **a internet conversa**. Este capítulo explica os tijolos que sustentam toda a web.

---

## 9.1 O que é uma API (revisão e aprofundamento)

No Capítulo 1 vimos que uma API é uma "porta de entrada" para sistemas conversarem. Agora vamos aprofundar.

### Definição mais técnica

> **API (Application Programming Interface)**: um conjunto de **regras e endereços padronizados** que permite que dois sistemas troquem informações de forma previsível.

Uma API expõe um conjunto de **endpoints** (URLs específicas), cada um aceitando determinados **métodos** e **dados**, e devolvendo determinadas **respostas**.

### API como contrato

Uma API funciona como um **contrato** entre quem consome e quem fornece um sistema.

Esse contrato responde perguntas como:

- Qual endereço devo chamar?
- Qual método HTTP devo usar?
- Quais dados preciso enviar?
- Quais dados vou receber?
- Quais erros podem acontecer?
- Preciso estar autenticado?
- Qual formato será usado: JSON, XML, arquivo, texto?

O consumidor da API não precisa saber se por dentro o backend usa C#, Java, Python, PostgreSQL, SQL Server, Redis ou uma fila de mensagens. Ele precisa saber apenas como conversar com a API.

Por isso documentação é tão importante. Em uma API real, o "cardápio" pode ser uma documentação escrita, uma coleção do Postman, ou uma especificação **Swagger/OpenAPI**.

### O que uma API não é

Para evitar confusão:

- API **não é o banco de dados**. Ela pode consultar o banco, mas não é o banco.
- API **não é a tela**. A tela consome a API.
- API **não é necessariamente REST**. REST é um estilo possível de API.
- API **não é só uma URL**. A URL sozinha não diz tudo; o endpoint completo envolve método, rota, parâmetros, headers, body e resposta esperada.

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

### REST com mais profundidade

Em REST, o foco não é criar URLs com nomes de ações, como `/criarProduto`, `/deletarProduto` ou `/buscarUsuario`. O foco é expor **recursos**:

- `/produtos`
- `/produtos/15`
- `/usuarios/42`
- `/usuarios/42/pedidos`

A ação vem do verbo HTTP:

| Ideia errada | Ideia REST |
|---|---|
| `GET /listarProdutos` | `GET /produtos` |
| `POST /criarProduto` | `POST /produtos` |
| `POST /deletarProduto/15` | `DELETE /produtos/15` |
| `POST /atualizarProduto/15` | `PUT /produtos/15` ou `PATCH /produtos/15` |

Isso deixa a API previsível. Quando você aprende o padrão de um recurso, consegue adivinhar o padrão dos outros.

### Stateless na prática

REST costuma ser **stateless**. Isso significa que cada requisição precisa carregar as informações necessárias para ser entendida.

Exemplo: se um endpoint exige autenticação, o cliente não deve depender de "memória" do servidor dizendo "esse usuário já passou aqui antes". Ele envia um token no header:

```http
Authorization: Bearer eyJhbGciOi...
```

O servidor recebe a requisição, valida o token, executa o necessário e responde. Na próxima requisição, o processo acontece de novo.

Isso facilita escalar uma API para vários servidores, porque qualquer servidor consegue atender a requisição sem depender de uma sessão guardada em memória local.

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

### HTTP vs HTTPS

**HTTP** define a conversa. **HTTPS** protege a conversa.

| Protocolo | O que significa | Como pensar |
|---|---|---|
| `http://` | HTTP puro, sem criptografia de transporte | A mensagem viaja legível pela rede |
| `https://` | HTTP sobre TLS | A mensagem viaja criptografada |

Quando uma API usa HTTPS, três coisas importantes acontecem:

1. **Criptografia**: quem interceptar o tráfego não consegue ler facilmente o conteúdo.
2. **Integridade**: se alguém alterar a mensagem no caminho, isso pode ser detectado.
3. **Autenticidade do servidor**: o certificado ajuda o cliente a saber que está falando com o servidor certo.

HTTPS não transforma automaticamente uma API ruim em segura. Ele protege o caminho entre cliente e servidor, mas a API ainda precisa validar entrada, proteger tokens, controlar permissões, evitar vazamento de dados e tratar erros corretamente.

Em desenvolvimento local é comum ver URLs como `https://localhost:5001`. Nesse caso, o certificado costuma ser de desenvolvimento. Em produção, o certificado precisa ser válido para o domínio real da API.

### O que acontece antes do backend receber a requisição?

Quando o cliente chama `https://api.exemplo.com/produtos/15`, existe um caminho antes do seu código C# executar:

1. O cliente descobre o IP do domínio via **DNS**.
2. O cliente abre uma conexão com o servidor.
3. Em HTTPS, cliente e servidor negociam uma conexão segura com **TLS**.
4. A requisição HTTP é enviada por essa conexão.
5. Alguma camada recebe a conexão: servidor web, proxy reverso, balanceador ou a própria aplicação.
6. O ASP.NET Core entra em ação e passa a requisição pelo pipeline.
7. O roteamento encontra o endpoint correto.
8. O controller/action executa.
9. O retorno vira uma resposta HTTP.
10. A resposta volta ao cliente pela mesma lógica.

Você não precisa configurar tudo isso manualmente no começo, mas precisa entender que backend profissional não é só "um método rodando": é uma aplicação recebendo tráfego real por uma cadeia de rede, segurança e processamento.

### Requisição no detalhe

Uma requisição pode carregar informações em vários lugares:

| Parte | Exemplo | Para que serve |
|---|---|---|
| Método | `GET` | Diz a intenção da chamada |
| Scheme | `https` | Diz se usa HTTP puro ou protegido |
| Host | `api.exemplo.com` | Diz qual servidor/domínio receberá a chamada |
| Path | `/produtos/15` | Identifica a rota/recurso |
| Query string | `?categoria=livros&page=2` | Envia filtros e opções de consulta |
| Headers | `Authorization`, `Accept`, `Content-Type` | Envia metadados |
| Body | `{ "nome": "Notebook" }` | Envia dados principais |

Exemplo juntando tudo:

```http
PUT /produtos/15?validarEstoque=true HTTP/1.1
Host: api.exemplo.com
Authorization: Bearer abc123
Accept: application/json
Content-Type: application/json

{
  "nome": "Notebook Pro",
  "preco": 6200.00
}
```

### Resposta no detalhe

Uma resposta também tem camadas:

| Parte | Exemplo | Para que serve |
|---|---|---|
| Status code | `200`, `201`, `404`, `500` | Resume o resultado |
| Reason phrase | `OK`, `Created`, `Not Found` | Descrição curta do status |
| Headers | `Content-Type`, `Location` | Metadados da resposta |
| Body | JSON, texto, arquivo | Conteúdo devolvido |

Exemplo:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 15,
  "nome": "Notebook Pro",
  "preco": 6200.00
}
```

Se der erro, a resposta também deve ser organizada:

```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "erro": "Produto não encontrado",
  "codigo": "PRODUTO_NAO_ENCONTRADO"
}
```

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

### Verbos HTTP em detalhes

Na prática de APIs REST, você usará `GET`, `POST`, `PUT`, `PATCH` e `DELETE` o tempo todo. Mas o HTTP tem outros métodos, e é bom reconhecer cada um:

| Verbo | É seguro? | É idempotente? | Body comum? | Uso em APIs |
|---|---|---|---|---|
| `GET` | Sim | Sim | Não | Consultar um recurso ou lista. Não deve alterar dados. |
| `POST` | Não | Não | Sim | Criar recurso, executar uma operação ou enviar dados para processamento. |
| `PUT` | Não | Sim | Sim | Substituir um recurso inteiro por uma nova versão. |
| `PATCH` | Não | Nem sempre | Sim | Alterar apenas alguns campos de um recurso. |
| `DELETE` | Não | Sim | Geralmente não | Remover um recurso. |
| `HEAD` | Sim | Sim | Não | Igual ao `GET`, mas sem body na resposta; útil para checar metadados. |
| `OPTIONS` | Sim | Sim | Não | Descobrir quais métodos/headers são permitidos; aparece muito em CORS. |
| `CONNECT` | Não | Não | Não | Usado para criar túnel de rede, comum em proxies. Quase nunca em APIs REST comuns. |
| `TRACE` | Sim | Sim | Não | Diagnóstico do caminho da requisição. Normalmente desabilitado por segurança. |

**Seguro**, aqui, não significa "protegido contra hackers". Significa que o método não deveria mudar dados no servidor. `GET`, `HEAD` e `OPTIONS` são considerados seguros porque servem para consultar.

### `PUT` vs `PATCH`

Essa diferença confunde muito:

| Situação | Melhor verbo |
|---|---|
| Cliente envia o objeto inteiro atualizado | `PUT` |
| Cliente envia só os campos que quer mudar | `PATCH` |

Exemplo com `PUT`:

```http
PUT /usuarios/42
Content-Type: application/json

{
  "nome": "Maria",
  "email": "maria@email.com",
  "ativo": true
}
```

Exemplo com `PATCH`:

```http
PATCH /usuarios/42
Content-Type: application/json

{
  "email": "novo-email@email.com"
}
```

Em projetos didáticos, muita gente usa `PUT` para atualização geral e deixa `PATCH` para depois. Em APIs profissionais, vale ser intencional: se a atualização é parcial, `PATCH` comunica melhor a intenção.

### Quando `POST` não significa apenas criar

O uso mais clássico de `POST` é criar:

```http
POST /produtos
```

Mas `POST` também pode representar uma operação que não se encaixa bem como CRUD simples:

```http
POST /pedidos/123/cancelamento
POST /relatorios/vendas/gerar
POST /login
```

Isso deve ser usado com cuidado. Se tudo virar `POST /fazerAlgumaCoisa`, a API perde a previsibilidade REST.

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

### Catálogo ampliado de status codes

Você não precisa decorar todos. Na prática, uma API comum usa muito `200`, `201`, `204`, `400`, `401`, `403`, `404`, `409`, `422` e `500`. Mas é importante reconhecer o mapa completo para entender logs, proxies, gateways, navegadores e integrações externas.

A IANA mantém o registro oficial dos status codes HTTP: <https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml>.

### 1xx — Informacionais

Esses códigos dizem: "recebi algo e ainda estou processando". Eles raramente aparecem diretamente no código da sua API.

| Code | Nome | Como entender |
|---|---|---|
| `100` | Continue | O cliente pode continuar enviando a requisição. Útil quando há body grande. |
| `101` | Switching Protocols | O servidor aceitou trocar de protocolo, por exemplo em upgrade de conexão. |
| `102` | Processing | O servidor recebeu e está processando, mas ainda não terminou. Muito associado a WebDAV. |
| `103` | Early Hints | Permite enviar dicas antecipadas, como recursos que o navegador pode começar a carregar. |
| `104` | Upload Resumption Supported | Código temporário para suporte a retomada de upload. É específico e pode mudar com a evolução da especificação. |
| `105-199` | Não atribuídos | Faixa livre no registro oficial. Não invente uso próprio em API pública. |

### 2xx — Sucesso

Esses códigos dizem: "a requisição foi aceita e deu certo de algum modo".

| Code | Nome | Quando aparece |
|---|---|---|
| `200` | OK | Sucesso geral. Muito usado em `GET`, `PUT`, `PATCH` e operações que retornam dados. |
| `201` | Created | Recurso criado com sucesso. Muito usado em `POST`. Normalmente pode vir com header `Location`. |
| `202` | Accepted | Pedido aceito, mas ainda não concluído. Útil para processamento assíncrono, filas e jobs. |
| `203` | Non-Authoritative Information | Resposta modificada por intermediário; raro em APIs comuns. |
| `204` | No Content | Sucesso sem body. Muito usado em `DELETE` ou `PUT` que não precisa devolver dados. |
| `205` | Reset Content | Sucesso e o cliente deveria limpar/reiniciar o formulário ou visualização. Raro. |
| `206` | Partial Content | Resposta parcial, comum em download, streaming ou requisições com `Range`. |
| `207` | Multi-Status | Retorna múltiplos resultados em uma resposta. Associado a WebDAV. |
| `208` | Already Reported | Evita repetir itens já reportados em respostas WebDAV. |
| `209-225` | Não atribuídos | Faixa livre. |
| `226` | IM Used | Resposta representa o resultado de manipulações de instância. Muito raro em APIs REST comuns. |
| `227-299` | Não atribuídos | Faixa livre. |

### 3xx — Redirecionamento

Esses códigos dizem: "para completar, vá para outro lugar" ou "use uma versão em cache".

| Code | Nome | Quando aparece |
|---|---|---|
| `300` | Multiple Choices | Existem várias opções de resposta e o cliente deve escolher. Raro. |
| `301` | Moved Permanently | O recurso mudou de endereço permanentemente. Navegadores e clientes podem guardar isso. |
| `302` | Found | Redirecionamento temporário tradicional. Em clientes antigos, pode mudar o método para `GET`. |
| `303` | See Other | Após uma operação, o cliente deve buscar o resultado em outra URL usando `GET`. |
| `304` | Not Modified | O recurso não mudou; o cliente pode usar cache. Muito usado com `ETag` e `If-None-Match`. |
| `305` | Use Proxy | Indica uso de proxy. Hoje é raro e pouco usado por questões de segurança. |
| `306` | Unused | Código reservado/sem uso atual. |
| `307` | Temporary Redirect | Redirecionamento temporário preservando método e body. Melhor definido que `302`. |
| `308` | Permanent Redirect | Redirecionamento permanente preservando método e body. |
| `309-399` | Não atribuídos | Faixa livre. |

Em APIs, redirecionamento deve ser usado com cuidado. Para mudança de versão, geralmente é melhor documentar `/api/v1` e `/api/v2` do que depender de redirecionamentos invisíveis.

### 4xx — Erro do cliente

Esses códigos dizem: "o servidor entendeu a requisição, mas o problema está no pedido enviado".

| Code | Nome | Quando usar ou quando aparece |
|---|---|---|
| `400` | Bad Request | Requisição inválida, JSON malformado, parâmetro errado, regra básica de entrada quebrada. |
| `401` | Unauthorized | Falta autenticação válida. Apesar do nome, significa "não autenticado". |
| `402` | Payment Required | Reservado/relacionado a pagamento. Pouco usado historicamente, mas aparece em APIs de cobrança. |
| `403` | Forbidden | Usuário autenticado, mas sem permissão para aquela ação. |
| `404` | Not Found | Recurso não encontrado ou rota inexistente. |
| `405` | Method Not Allowed | A rota existe, mas não aceita aquele verbo. Ex: `POST` em endpoint que só aceita `GET`. |
| `406` | Not Acceptable | A API não consegue responder no formato pedido pelo header `Accept`. |
| `407` | Proxy Authentication Required | O cliente precisa se autenticar em um proxy. |
| `408` | Request Timeout | O cliente demorou demais para enviar a requisição. |
| `409` | Conflict | Conflito com o estado atual. Ex: e-mail duplicado, versão desatualizada, recurso já existe. |
| `410` | Gone | O recurso existia, mas foi removido permanentemente. Diferente de um `404` genérico. |
| `411` | Length Required | O servidor exige `Content-Length` e o cliente não enviou. |
| `412` | Precondition Failed | Uma condição enviada em headers como `If-Match` falhou. Útil para controle de concorrência. |
| `413` | Content Too Large | Body grande demais. Ex: upload maior que o limite. |
| `414` | URI Too Long | URL grande demais, comum quando se coloca informação demais na query string. |
| `415` | Unsupported Media Type | `Content-Type` não suportado. Ex: cliente manda XML onde a API só aceita JSON. |
| `416` | Range Not Satisfiable | Cliente pediu uma faixa de bytes inválida. |
| `417` | Expectation Failed | O servidor não conseguiu cumprir a expectativa indicada pelo header `Expect`. |
| `418` | Unused | Código sem uso atual no registro oficial. Você ainda pode ver referências históricas/brincadeiras a ele. |
| `419-420` | Não atribuídos | Faixa livre. Alguns frameworks usam códigos não oficiais aqui, mas não é padrão HTTP. |
| `421` | Misdirected Request | A requisição foi enviada para um servidor que não consegue produzir resposta para aquele destino. |
| `422` | Unprocessable Content | O formato é válido, mas o conteúdo falha em regras semânticas. Muito usado para validação. Você também verá o nome antigo `Unprocessable Entity`. |
| `423` | Locked | Recurso bloqueado. Associado a WebDAV. |
| `424` | Failed Dependency | Falha porque uma operação dependente falhou. Associado a WebDAV. |
| `425` | Too Early | Servidor não quer processar algo que pode ser repetido cedo demais em conexão TLS. Raro em APIs comuns. |
| `426` | Upgrade Required | Cliente precisa mudar/atualizar protocolo. |
| `427` | Não atribuído | Faixa livre. |
| `428` | Precondition Required | Servidor exige uma condição, como `If-Match`, para evitar atualização perdida. |
| `429` | Too Many Requests | Cliente fez requisições demais. Usado em rate limiting. |
| `430` | Não atribuído | Faixa livre. |
| `431` | Request Header Fields Too Large | Headers grandes demais. Pode acontecer com cookies ou tokens enormes. |
| `432-450` | Não atribuídos | Faixa livre. |
| `451` | Unavailable For Legal Reasons | Conteúdo indisponível por motivo legal. |
| `452-499` | Não atribuídos | Faixa livre. Alguns servidores usam códigos não oficiais nessa faixa, como `499`, mas não são padrão HTTP. |

### 5xx — Erro do servidor

Esses códigos dizem: "a requisição parecia válida, mas o servidor ou algum intermediário falhou".

| Code | Nome | Quando usar ou quando aparece |
|---|---|---|
| `500` | Internal Server Error | Erro inesperado no backend. É o "algo quebrou" genérico. |
| `501` | Not Implemented | Servidor não suporta a funcionalidade necessária para atender a requisição. |
| `502` | Bad Gateway | Um gateway/proxy recebeu resposta inválida de outro servidor. Comum em arquiteturas com proxy reverso. |
| `503` | Service Unavailable | Serviço temporariamente indisponível: manutenção, sobrecarga, deploy, dependência fora. |
| `504` | Gateway Timeout | Gateway/proxy esperou resposta de outro servidor e o tempo acabou. |
| `505` | HTTP Version Not Supported | Versão HTTP da requisição não suportada. |
| `506` | Variant Also Negotiates | Erro de negociação de conteúdo no servidor. Raro. |
| `507` | Insufficient Storage | Servidor não tem armazenamento suficiente para completar a operação. Associado a WebDAV. |
| `508` | Loop Detected | Servidor detectou loop infinito ao processar a requisição. Associado a WebDAV. |
| `509` | Não atribuído | Faixa livre. |
| `510` | Not Extended | Código obsoleto ligado a extensões HTTP antigas. |
| `511` | Network Authentication Required | Cliente precisa autenticar na rede, comum em portais cativos de Wi-Fi. |
| `512-599` | Não atribuídos | Faixa livre. |

### Como escolher o status code certo em uma API

Use esta cola mental:

| Situação | Status recomendado |
|---|---|
| Buscou e encontrou | `200 OK` |
| Criou um novo recurso | `201 Created` |
| Deu certo, mas não há nada para devolver | `204 No Content` |
| JSON ou dados básicos inválidos | `400 Bad Request` |
| Não enviou token/login válido | `401 Unauthorized` |
| Está logado, mas não tem permissão | `403 Forbidden` |
| Recurso não existe | `404 Not Found` |
| Verbo errado para aquela rota | `405 Method Not Allowed` |
| Conflito com dado existente | `409 Conflict` |
| Validação de regra de negócio falhou | `422 Unprocessable Content` |
| Cliente passou do limite de chamadas | `429 Too Many Requests` |
| Erro inesperado no seu código | `500 Internal Server Error` |
| Dependência externa/proxy falhou | `502 Bad Gateway` ou `504 Gateway Timeout` |
| API temporariamente fora | `503 Service Unavailable` |

Regra de ouro: **status code não é decoração**. Ele é parte do contrato da API. Um frontend, um app mobile ou outro backend pode tomar decisões automáticas com base nele.

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

### URL, rota, recurso e endpoint

Essas palavras aparecem juntas, mas não são exatamente a mesma coisa:

| Termo | Exemplo | O que significa |
|---|---|---|
| **URL completa** | `https://api.exemplo.com/produtos/15?incluirEstoque=true` | Endereço inteiro chamado pelo cliente. |
| **Host/domínio** | `api.exemplo.com` | Servidor que receberá a chamada. |
| **Rota/path** | `/produtos/15` | Caminho dentro da API. |
| **Query string** | `?incluirEstoque=true` | Filtros/opções adicionais. |
| **Recurso** | `produtos` | A "coisa" principal exposta pela API. |
| **Endpoint** | `GET /produtos/15` | Combinação de método HTTP + rota. |

Dois endpoints podem ter a mesma rota e métodos diferentes:

```http
GET    /produtos/15
PUT    /produtos/15
DELETE /produtos/15
```

A rota é a mesma, mas a intenção muda completamente por causa do verbo.

### Endpoints devem expressar recursos, não telas

Uma API não precisa copiar os nomes das páginas do frontend.

Exemplo: uma tela chamada "Dashboard do Administrador" talvez consuma vários endpoints:

```http
GET /usuarios?ativos=true
GET /pedidos?status=pendente
GET /relatorios/vendas?periodo=mensal
```

O endpoint deve representar uma capacidade do sistema, não necessariamente uma tela específica.

### Rotas boas vs rotas confusas

| Evite | Prefira | Motivo |
|---|---|---|
| `/getAllProducts` | `GET /produtos` | O verbo `GET` já diz que é busca. |
| `/produto/15` | `/produtos/15` | Recursos REST geralmente usam plural. |
| `/deletar/15` | `DELETE /produtos/15` | A ação deve ficar no verbo HTTP. |
| `/usuarios/42/listar-pedidos` | `GET /usuarios/42/pedidos` | Pedidos são recurso relacionado ao usuário. |
| `/api/produtos?id=15` | `/api/produtos/15` | Identificador principal costuma ficar na rota. |

### Endpoint também tem contrato de entrada e saída

Quando você documenta um endpoint, não basta dizer a URL. O contrato completo deveria responder:

| Pergunta | Exemplo |
|---|---|
| Qual método? | `POST` |
| Qual rota? | `/produtos` |
| Precisa autenticação? | Sim, JWT no header `Authorization` |
| Quais headers? | `Content-Type: application/json` |
| Quais parâmetros de rota? | Nenhum |
| Quais query params? | Nenhum |
| Qual body? | `{ "nome": "Notebook", "preco": 4500 }` |
| Quais respostas possíveis? | `201`, `400`, `401`, `409`, `500` |
| Qual JSON de resposta? | Produto criado com `id` |

Essa visão evita uma API que "funciona no meu Swagger", mas ninguém sabe consumir direito.

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
- **Swagger / OpenAPI** (vem **embutido** em projetos ASP.NET Core).

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
6. **Documente**: use **Swagger/OpenAPI** (vem grátis no ASP.NET Core).
7. **Valide entradas**: nunca confie no que o cliente manda.
8. **Pagine listas grandes**: nunca devolva 1 milhão de itens de uma vez.
9. **HTTPS sempre**, em produção. Nunca HTTP puro.
10. **Não exponha dados sensíveis** (senha, hash, dados internos do banco) em respostas.

---

## 9.11 Resumo do capítulo

- API REST é o estilo mais usado para APIs hoje.
- API é um **contrato**: método, rota, dados de entrada, respostas possíveis e regras de acesso.
- HTTP é o protocolo: cada interação é **requisição → resposta**.
- HTTPS é HTTP protegido por TLS: criptografia, integridade e autenticidade do servidor.
- **Verbos** indicam intenção: `GET`, `POST`, `PUT`, `DELETE`...
- Verbos têm propriedades importantes: alguns são seguros, alguns são idempotentes.
- **Status codes** comunicam o resultado (200, 201, 400, 404, 500...).
- **JSON** é o formato padrão de troca de dados.
- **Endpoints** = verbo + rota. Existem 4 tipos de parâmetros: **route, query, body, header**.
- Uma boa rota REST representa recursos, não nomes de telas nem nomes de funções.

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
   - Usuário não enviou token.
   - Usuário está logado, mas não é administrador.
   - Produto já existe com o mesmo nome.
   - Cliente fez chamadas demais em pouco tempo.
   - Upload passou do tamanho máximo.
3. Quando usar **query** vs **route** vs **body**? Dê 1 exemplo de cada.
4. Escreva um JSON representando uma "tarefa" com `id`, `titulo`, `concluida`, `prazo`.
5. Explique a diferença entre HTTP e HTTPS sem usar termos técnicos demais.
6. Para cada rota ruim, escreva uma versão REST melhor:
   - `GET /listarProdutos`
   - `POST /criarUsuario`
   - `POST /deletarPedido/8`
   - `GET /cliente/5/listarPedidos`
7. Monte o contrato de um endpoint `POST /clientes`: método, rota, headers, body esperado, status de sucesso e possíveis erros.
8. Explique a diferença entre `PUT /usuarios/10` e `PATCH /usuarios/10`.
9. Por que retornar `200 OK` quando aconteceu erro atrapalha quem consome a API?

---

➡️ **Próximo capítulo:** [Capítulo 10 — Construindo uma API REST com ASP.NET Core](10-Construindo-API-REST-ASPNET-Core.md)
