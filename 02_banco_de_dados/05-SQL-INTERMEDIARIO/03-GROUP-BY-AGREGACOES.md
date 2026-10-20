# `GROUP BY` e agregações

Funções de agregação resumem várias linhas em um único valor. Sem `GROUP BY`, a função resume toda a tabela; com `GROUP BY`, o PostgreSQL separa as linhas em grupos e calcula um resultado para cada grupo.

Quando uma consulta mistura colunas comuns com agregações, toda coluna comum exibida no `SELECT` precisa estar no `GROUP BY` (ou também ser calculada por uma agregação). Essa regra garante que cada linha do resultado represente um grupo bem definido.

As agregações também tratam `NULL` de forma importante: em geral, `SUM`, `AVG`, `MIN`, `MAX` e `COUNT(coluna)` ignoram valores nulos. Já `COUNT(*)` conta a linha mesmo que algumas de suas colunas estejam nulas. Saber essa diferença evita contagens e médias incorretas.

## `COUNT`
Conta linhas ou valores. `COUNT(*)` conta as linhas do resultado, enquanto `COUNT(nome_da_coluna)` ignora valores `NULL` nessa coluna.

No exemplo, não há `GROUP BY`, então a consulta devolve uma única linha contendo o número total de linhas de `clientes`. Se a tabela estiver vazia, `COUNT(*)` retorna `0`.
```sql
SELECT COUNT(*) FROM clientes;
```

## `SUM`
Soma os valores de uma coluna numérica, como o estoque total de todos os produtos.

O resultado representa a soma de todos os valores não nulos de `estoque`. Ele não conta produtos: ele acumula a quantidade disponível. Para somar por categoria, por exemplo, seria necessário agrupar pelas categorias antes de aplicar `SUM`.
```sql
SELECT SUM(estoque) FROM produtos;
```

## `AVG`
Calcula a média dos valores numéricos. O resultado pode ter casas decimais mesmo quando os dados armazenados parecem inteiros.

No exemplo, a média é calculada sobre todos os preços cadastrados. Ela é uma medida geral do catálogo e não indica necessariamente o preço de um produto específico; valores muito altos ou muito baixos influenciam o resultado.
```sql
SELECT AVG(preco) FROM produtos;
```

## `MIN`
Retorna o menor valor encontrado, útil para descobrir o menor preço ou a data mais antiga.

Como a coluna consultada é `preco`, o resultado é o preço mais baixo da tabela `produtos`. A função não informa qual produto possui esse preço; para isso, seria necessário consultar também a linha do produto correspondente.
```sql
SELECT MIN(preco) FROM produtos;
```

## `MAX`
Retorna o maior valor encontrado, como o produto mais caro ou o maior estoque.

Neste caso, o resultado é o maior preço cadastrado. Assim como `MIN`, `MAX` devolve apenas o valor agregado, não os outros dados da linha que o contém.
```sql
SELECT MAX(preco) FROM produtos;
```

## Pedidos por cliente
Neste agrupamento, cada cliente forma um grupo e `COUNT` calcula quantos pedidos pertencem a ele. O `LEFT JOIN` garante que também apareçam clientes sem pedidos, com quantidade igual a zero.

As duas colunas listadas em `GROUP BY` identificam o grupo que será exibido. O banco reúne todas as linhas com o mesmo cliente e aplica o `COUNT` dentro desse conjunto, produzindo uma linha final para cada cliente.
```sql
SELECT
    c.id_cliente,
    c.nome,
    COUNT(p.id_pedido) AS quantidade_pedidos
FROM clientes c
LEFT JOIN pedidos p
    ON p.id_cliente = c.id_cliente
GROUP BY
    c.id_cliente,
    c.nome;
```

Em `LEFT JOIN`, `COUNT(p.id_pedido)` ignora `NULL` e é melhor para contar pedidos realmente existentes.

Isso é diferente de `COUNT(*)`: para um cliente sem pedido, o `LEFT JOIN` ainda produz uma linha, mas `p.id_pedido` fica `NULL`. Ao contar o identificador do pedido, essa linha não é confundida com um pedido real.

Também é possível que um cliente apareça em vários pedidos. Nesse caso, cada pedido ocupa uma linha antes do agrupamento, e `COUNT(p.id_pedido)` reúne essas linhas em uma única contagem para o cliente.
