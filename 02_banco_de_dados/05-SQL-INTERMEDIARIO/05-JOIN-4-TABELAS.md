# JOIN com quatro tabelas

Esta consulta percorre o relacionamento completo de uma venda: `pedidos` identifica a compra, `clientes` identifica quem comprou, `itens_pedido` lista os itens e `produtos` fornece o nome e os dados do produto. Cada condição `ON` conecta uma chave estrangeira à chave correspondente da próxima tabela.

Como um pedido pode ter vários itens, o resultado normalmente terá uma linha por item, e não uma linha por pedido. O `subtotal` é calculado na consulta multiplicando a quantidade pelo preço unitário registrado no item.

Os apelidos usados na consulta tornam a origem de cada coluna visível: `p` representa `pedidos`, `c` representa `clientes`, `i` representa `itens_pedido` e `pr` representa `produtos`. Isso é especialmente útil quando várias tabelas possuem colunas com nomes parecidos, como identificadores e datas.

A consulta começa em `pedidos` e segue as ligações naturais do modelo: o pedido aponta para o cliente por `id_cliente`; os itens apontam para o pedido por `id_pedido`; e cada item aponta para seu produto por `id_produto`. Cada `ON` deve comparar as colunas que representam essa mesma relação.

```sql
SELECT
    p.id_pedido,
    c.nome AS cliente,
    c.sobrenome,
    pr.nome AS produto,
    i.quantidade,
    i.preco_unit,
    i.quantidade * i.preco_unit AS subtotal
FROM pedidos p
INNER JOIN clientes c
    ON c.id_cliente = p.id_cliente
INNER JOIN itens_pedido i
    ON i.id_pedido = p.id_pedido
INNER JOIN produtos pr
    ON pr.id_produto = i.id_produto
ORDER BY
    p.id_pedido,
    pr.nome;
```

O `SELECT` escolhe os dados que serão mostrados. `AS cliente` e `AS produto` definem nomes mais claros para as colunas finais; os dados continuam vindo de `c.nome` e `pr.nome`. A expressão `i.quantidade * i.preco_unit` é calculada para cada item e recebe o nome `subtotal`.

`ORDER BY p.id_pedido, pr.nome` organiza primeiro os itens pelo pedido e, dentro de cada pedido, pelo nome do produto. A ordenação melhora a leitura do relatório, mas não altera quais linhas foram encontradas.

## Por que pode retornar nada?
Porque `INNER JOIN` exige correspondência em todas as tabelas.

Uma linha só chega ao resultado se houver cliente para o pedido, item para o pedido e produto para o item.

Se `pedidos` ou `itens_pedido` estiverem vazios, o resultado será vazio.

O mesmo pode acontecer com uma chave estrangeira incorreta, com um item apontando para um produto inexistente ou quando a consulta é executada antes da carga dos dados. O problema não está necessariamente no `SELECT`; pode estar na ausência de dados relacionados.

Como há vários `INNER JOIN`, uma falha em qualquer elo interrompe a linha inteira. Por exemplo, mesmo que o pedido e o cliente existam, o pedido não aparecerá se não possuir itens. Da mesma forma, um item não será exibido se não houver produto correspondente.

## Diagnóstico
Troque temporariamente por `LEFT JOIN` para enxergar onde aparecem `NULL`.

Comece pela tabela principal (`pedidos`) e examine as colunas da tabela seguinte; a primeira coluna nula indica em qual relacionamento a cadeia deixou de encontrar correspondência.

Faça o diagnóstico em etapas: confirme se há pedidos, depois se cada pedido encontra um cliente, depois se encontra itens e, por fim, se os itens encontram produtos. Essa sequência acompanha exatamente a ordem dos relacionamentos mostrados na consulta.

Depois de identificar o ponto do problema, confira os valores das chaves (`id_pedido`, `id_cliente` e `id_produto`) e restaure `INNER JOIN` se a consulta final deve mostrar somente vendas completas.

O preço usado no subtotal vem de `itens_pedido.preco_unit`, e não de `produtos.preco`. Isso permite que o relatório preserve o valor cobrado no momento do pedido, mesmo que o preço atual do produto seja alterado depois.
