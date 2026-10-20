# INNER, LEFT e RIGHT JOIN — passo a passo

Para entender um `JOIN`, imagine que o PostgreSQL compara o valor da chave de cada linha com a condição indicada em `ON`. Quando um cliente possui dois pedidos, a linha desse cliente é combinada duas vezes; isso explica por que um `JOIN` pode aumentar a quantidade de linhas do resultado.

Considere:

As tabelas abaixo representam um relacionamento simples: `clientes.id` identifica cada cliente e `pedidos.cliente` guarda o identificador do cliente que fez o pedido. O valor repetido `1` mostra que um cliente pode ter vários pedidos.

Esse é um relacionamento de um para muitos: uma linha de `clientes` pode estar ligada a várias linhas de `pedidos`, mas cada pedido pertence a apenas um cliente. Quando o `JOIN` encontra duas linhas de pedido para Ana, ele repete os dados de Ana em duas linhas do resultado para representar as duas relações.

## clientes
|  id | nome  |
| --: | ----- |
|   1 | Ana   |
|   2 | Bruno |
|   3 | Carla |

## pedidos
| pedido | cliente |
| -----: | ------: |
|    100 |       1 |
|    101 |       1 |
|    102 |       2 |

## INNER
Resultado:
```text
Ana   100
Ana   101
Bruno 102
```

Carla não aparece.

O `INNER JOIN` elimina qualquer linha que não consiga encontrar uma correspondência dos dois lados. Como Carla não possui pedido, não há uma combinação válida para ela no resultado.

Repare que o `INNER JOIN` não está dizendo que Carla foi apagada da tabela `clientes`. Ela apenas não atende à condição do resultado dessa consulta específica. Ao consultar somente `clientes`, Carla continua existindo normalmente.

## LEFT
Resultado:
```text
Ana   100
Ana   101
Bruno 102
Carla NULL
```

O `LEFT JOIN` começa pelas linhas de `clientes` e tenta encontrar pedidos para cada uma. Quando não encontra pedido para Carla, ele mantém Carla e preenche as colunas da tabela `pedidos` com `NULL`.

O lado que define a preservação é a posição, e não o nome da tabela. Se a consulta fosse escrita começando por `pedidos LEFT JOIN clientes`, todos os pedidos seriam preservados e um cliente sem pedido, como Carla, não apareceria.

`NULL` sinaliza que não foi possível encontrar uma linha relacionada. Ele não é um pedido com número zero; é a ausência de pedido para aquela combinação de dados.

## RIGHT

O `RIGHT JOIN` preserva a tabela colocada à direita da cláusula. Ele pode ser entendido como um `LEFT JOIN` escrito com as tabelas em ordem inversa. Para decidir qual usar, observe qual tabela precisa continuar no resultado quando não houver ligação entre as duas.

## Clientes sem pedidos
```sql
SELECT c.*
FROM clientes c
LEFT JOIN pedidos p
    ON p.id_cliente = c.id_cliente
WHERE p.id_pedido IS NULL;
```

Esse padrão localiza registros sem relacionamento: o `LEFT JOIN` preserva todos os clientes, e o `WHERE` seleciona somente aqueles para os quais a coluna do lado direito permaneceu `NULL`. É importante testar uma coluna que não possa ser nula quando existe um pedido, como o identificador.

O filtro vem depois do `JOIN` porque ele analisa o resultado que já contém os valores nulos produzidos pelo relacionamento. Se fosse usada uma condição comum sobre `p` no `WHERE`, como um status de pedido, clientes sem pedido poderiam desaparecer; esse é um detalhe importante ao combinar filtros com `LEFT JOIN`.
