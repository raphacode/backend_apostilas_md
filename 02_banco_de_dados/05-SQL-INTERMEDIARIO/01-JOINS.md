# JOINs

Um `JOIN` combina linhas de duas ou mais tabelas usando uma condição de relacionamento. Em um banco relacional, é comum guardar clientes e pedidos em tabelas separadas e usar a chave estrangeira do pedido para recuperar os dados do cliente.

O tipo de `JOIN` define quais linhas serão preservadas quando não existir correspondência. A condição escrita depois de `ON` informa ao PostgreSQL como as tabelas devem ser relacionadas; ela não é um filtro de texto, mas a regra que liga as chaves.

Nos exemplos, `p` é um apelido para `pedidos` e `c` é um apelido para `clientes`. Os apelidos deixam a consulta menor e evitam dúvida sobre a origem de uma coluna: `p.id_cliente`, por exemplo, é a coluna `id_cliente` da tabela `pedidos`.

Em geral, `pedidos.id_cliente` guarda a chave estrangeira que aponta para `clientes.id_cliente`. A condição `c.id_cliente = p.id_cliente` compara esses dois valores. Quando o valor é igual, o PostgreSQL pode montar uma linha com dados do pedido e do cliente.

## `INNER JOIN`
Mostra apenas correspondências.

Isso significa que somente as linhas que possuem correspondência nas duas tabelas são exibidas. É útil quando o resultado precisa conter apenas registros relacionados, como pedidos que realmente possuem um cliente válido.

No exemplo, a tabela base é `pedidos`. Para cada pedido, o banco procura um cliente cujo identificador seja igual ao `id_cliente` armazenado no pedido. Se não encontrar, aquele pedido não entra no resultado; do mesmo modo, clientes sem pedido também não aparecem.

O `INNER JOIN` não cria uma coluna nova nem altera as tabelas. Ele monta um resultado temporário para aquela consulta. Por isso, é seguro usá-lo para consultar relacionamentos sem modificar os dados gravados no banco.

```sql
SELECT
    p.id_pedido,
    c.nome
FROM pedidos p
INNER JOIN clientes c
    ON c.id_cliente = p.id_cliente;
```

## `LEFT JOIN`
Mantém todas as linhas da tabela esquerda.

Ele acrescenta os dados correspondentes da tabela da direita. Quando não encontra correspondência, as colunas da direita recebem `NULL`.

Neste caso, `clientes` é a tabela da esquerda, pois aparece antes de `LEFT JOIN`. O banco parte de cada cliente e tenta encontrar pedidos dele. Isso torna o `LEFT JOIN` apropriado para relatórios que precisam listar todos os clientes, inclusive os que ainda não possuem movimentação.

```sql
SELECT
    c.id_cliente,
    c.nome,
    p.id_pedido
FROM clientes c
LEFT JOIN pedidos p
    ON p.id_cliente = c.id_cliente;
```

Cliente sem pedido aparece com `NULL`.

Neste exemplo, todos os clientes aparecem, inclusive os que ainda não fizeram pedidos. Se um cliente tiver vários pedidos, ele aparecerá em várias linhas, uma para cada correspondência.

O valor `NULL` não significa texto vazio nem número zero: ele representa informação ausente ou desconhecida. Ao analisar um resultado com `LEFT JOIN`, use testes como `IS NULL` ou `IS NOT NULL` para verificar essa situação; comparações com `= NULL` não produzem o resultado esperado em SQL.

## `RIGHT JOIN`
Mantém todas as linhas da tabela direita.

Ele preserva a tabela que aparece à direita do `JOIN` e produz uma ideia equivalente à de inverter a ordem das tabelas e usar `LEFT JOIN`; por isso, muitos times preferem `LEFT JOIN` para deixar a leitura mais direta.

No exemplo, a consulta começa por `pedidos`, mas `clientes` está no lado direito. Assim, um cliente sem pedido continua aparecendo, enquanto um pedido sem cliente não é preservado. O efeito é o mesmo de começar por `clientes` e escrever `LEFT JOIN pedidos`.

```sql
SELECT
    c.nome,
    p.id_pedido
FROM pedidos p
RIGHT JOIN clientes c
    ON p.id_cliente = c.id_cliente;
```

Aqui, `clientes` está à direita e, por isso, todos os clientes são preservados. A escolha entre `LEFT` e `RIGHT` não muda a relação entre as chaves: muda apenas qual lado deve continuar aparecendo mesmo sem correspondência.

Ao escolher um tipo de `JOIN`, comece definindo qual entidade não pode desaparecer do relatório. Se todos os clientes devem aparecer, use `clientes` como lado preservado; se só interessam relações completas, use `INNER JOIN`.
