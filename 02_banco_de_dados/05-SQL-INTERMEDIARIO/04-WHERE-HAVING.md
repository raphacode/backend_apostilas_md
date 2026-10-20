# `WHERE` x `HAVING`

`WHERE` e `HAVING` filtram em momentos diferentes da consulta. A ordem lógica é: selecionar as tabelas, relacionar as linhas, aplicar `WHERE`, formar os grupos com `GROUP BY`, calcular as agregações e só então aplicar `HAVING`.

Essa ordem explica por que cada cláusula tem uma responsabilidade própria. Antes de existir um grupo, não há como testar uma contagem de pedidos; depois que os grupos foram formados, um filtro de status aplicado tardiamente já não reduz as linhas que participam da contagem.

## `WHERE`
Filtra linhas antes do agrupamento.

Ele filtra linhas individuais antes do agrupamento. Use-o para reduzir os registros que participarão da contagem, soma ou outra agregação.

No trecho apresentado, cada linha de `pedidos` é avaliada separadamente. Somente pedidos cujo `status` seja `pago` seguem para as próximas etapas da consulta; pedidos abertos, enviados ou cancelados ficam fora da contagem.

```sql
WHERE p.status = 'pago'
```

## `HAVING`
Filtra grupos após `GROUP BY`.

Ele é indicado quando a condição depende de uma função de agregação, como manter apenas clientes com pelo menos dois pedidos.

Neste caso, o banco precisa primeiro calcular `COUNT(p.id_pedido)` para cada cliente. Só depois ele consegue decidir se a quantidade atende à condição `>= 2`.

```sql
HAVING COUNT(p.id_pedido) >= 2
```

## Exemplo
Neste exemplo, primeiro entram apenas pedidos com status `pago`. Depois, esses pedidos são agrupados por cliente e o `HAVING` mantém somente os grupos cuja contagem seja igual ou superior a dois.

O `INNER JOIN` garante que cada pedido pago seja associado ao seu cliente. Em seguida, `GROUP BY c.id_cliente, c.nome` cria um grupo por cliente. O nome `pedidos_pagos`, dado com `AS`, é apenas o rótulo da coluna exibida no resultado.
```sql
SELECT
    c.id_cliente,
    c.nome,
    COUNT(p.id_pedido) AS pedidos_pagos
FROM clientes c
INNER JOIN pedidos p
    ON p.id_cliente = c.id_cliente
WHERE p.status = 'pago'
GROUP BY
    c.id_cliente,
    c.nome
HAVING COUNT(p.id_pedido) >= 2;
```

Uma condição sobre uma coluna de cada pedido, como `p.status = 'pago'`, pertence ao `WHERE`. Uma condição sobre o resultado do grupo, como `COUNT(p.id_pedido) >= 2`, pertence ao `HAVING`; trocar os dois pode gerar erro ou alterar o resultado.

Uma forma de lembrar é: `WHERE` pergunta “esta linha entra no cálculo?” e `HAVING` pergunta “este grupo fica no resultado?”. Essa separação também deixa as consultas mais fáceis de ler e manter.
