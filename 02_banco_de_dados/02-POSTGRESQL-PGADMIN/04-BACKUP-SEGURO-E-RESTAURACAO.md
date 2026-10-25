# Backup seguro e restauração no PostgreSQL

## Objetivos da aula

Ao final desta aula, você conseguirá:

- explicar por que um backup é necessário antes de uma alteração importante;
- criar um backup de um banco PostgreSQL pelo pgAdmin;
- reconhecer a diferença entre os formatos `Custom` e `Plain`;
- restaurar um backup em um banco de teste;
- conferir se a restauração trouxe tabelas e dados corretamente.

> [!important] Regra de segurança
> Faça os testes em um banco de estudo. Antes de restaurar, apagar ou substituir qualquer dado, confirme o nome do banco conectado e mantenha uma cópia do backup original.

## 1. O que é backup?

Backup é uma cópia dos dados que pode ser usada para recuperar o banco depois de um erro, uma exclusão acidental, uma falha no computador ou uma alteração que não deveria ter sido feita.

Um backup não é apenas “salvar um arquivo”. Para ser útil, ele precisa poder ser localizado e restaurado. Por isso, guarde-o em um local conhecido e use nomes claros.

Exemplo de nome de arquivo:

```text
loja_2026-10-27.backup
```

## 2. Antes de começar

Confira estes itens:

- você está conectado ao banco correto;
- o banco é de teste ou você recebeu autorização para trabalhar nele;
- há espaço disponível para salvar o arquivo;
- o nome do arquivo tem o nome do banco e a data;
- você sabe em qual pasta o backup será salvo.

No pgAdmin, observe o nome da conexão no topo da Query Tool. Uma aba pode continuar ligada a outro banco mesmo que você clique em um nome diferente na árvore lateral.

## 3. Formatos de backup

### Custom (`.backup`)

O formato `Custom` é indicado para a maior parte dos exercícios no pgAdmin.

- gera um arquivo próprio do PostgreSQL;
- permite restaurar tabelas e dados com o pgAdmin ou `pg_restore`;
- facilita selecionar partes da restauração quando necessário.

Exemplo:

```text
loja_2026-10-27.backup
```

### Plain (`.sql`)

O formato `Plain` gera um arquivo de texto com comandos SQL.

- pode ser aberto e lido em um editor;
- mostra instruções como `CREATE TABLE` e `INSERT`;
- pode ser executado por ferramentas SQL, com cuidado para não aplicar os comandos no banco errado.

Exemplo:

```text
loja_2026-10-27.sql
```

## 4. Criando um backup no pgAdmin

1. Na árvore do pgAdmin, localize o banco que será copiado.
2. Clique com o botão direito no nome do banco.
3. Escolha **Backup...**.
4. Em **Filename**, selecione uma pasta e informe um nome claro, por exemplo `loja_2026-10-27.backup`.
5. Em **Format**, escolha `Custom`.
6. Confira mais uma vez o banco selecionado e o caminho do arquivo.
7. Clique em **Backup**.
8. Aguarde a mensagem de conclusão e confirme que o arquivo existe na pasta escolhida.

> [!tip] Boa prática
> Não use nomes genéricos como `backup.backup`. Um nome com banco e data evita confusão quando houver várias cópias.

## 5. Criando o backup pelo terminal

O PostgreSQL também oferece comandos para backup. No terminal, o comando `pg_dump` cria a cópia do banco.

### Backup no formato Custom

```powershell
pg_dump -U postgres -F c -f "C:\backups\loja_2026-10-27.backup" loja
```

Significado dos principais trechos:

- `-U postgres`: usuário usado na conexão;
- `-F c`: formato `Custom`;
- `-f`: caminho e nome do arquivo que será criado;
- `loja`: nome do banco que será copiado.

### Backup no formato SQL

```powershell
pg_dump -U postgres -F p -f "C:\backups\loja_2026-10-27.sql" loja
```

No formato `Plain`, `-F p` produz um arquivo SQL legível.

> [!warning] Atenção
> Os comandos de terminal devem apontar para o banco e para a pasta corretos. Leia o comando inteiro antes de executá-lo.

## 6. Restaurando com segurança

Nunca use a primeira restauração para substituir o banco que você quer preservar. Crie um banco de teste para conferir o backup.

### Criar o banco de teste

No pgAdmin, crie um banco chamado, por exemplo:

```text
loja_restaurada_teste
```

Também é possível usar SQL:

```sql
CREATE DATABASE loja_restaurada_teste;
```

### Restaurar pelo pgAdmin

1. Clique com o botão direito no banco de teste.
2. Escolha **Restore...**.
3. Selecione o arquivo `.backup` criado anteriormente.
4. Confira que o banco de destino é `loja_restaurada_teste`.
5. Clique em **Restore**.
6. Atualize a árvore do banco para visualizar tabelas, esquemas e dados restaurados.

### Restaurar pelo terminal

Para um arquivo no formato `Custom`:

```powershell
pg_restore -U postgres -d loja_restaurada_teste "C:\backups\loja_2026-10-27.backup"
```

Para um arquivo SQL no formato `Plain`:

```powershell
psql -U postgres -d loja_restaurada_teste -f "C:\backups\loja_2026-10-27.sql"
```

## 7. Como conferir a restauração

Depois de restaurar, abra a Query Tool conectada ao banco de teste e confira as tabelas:

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY table_name;
```

Em seguida, confira uma tabela conhecida. Adapte o nome para a tabela do seu banco:

```sql
SELECT COUNT(*) AS total_clientes
FROM clientes;
```

Compare o resultado com o banco original. Se as tabelas e as quantidades esperadas aparecerem, a restauração funcionou.

## 8. Checklist final

- [ ] Confirmei o nome do banco antes do backup.
- [ ] Salvei o arquivo com nome e data claros.
- [ ] Verifiquei se o arquivo foi criado na pasta escolhida.
- [ ] Criei um banco de teste para a restauração.
- [ ] Restaurei sem substituir o banco original.
- [ ] Conferi tabelas e dados no banco restaurado.

## Exercício prático

1. Escolha um banco de teste usado nas aulas.
2. Gere um backup no formato `Custom`.
3. Crie um banco com o sufixo `_restaurada_teste`.
4. Restaure o arquivo nesse banco de teste.
5. Execute uma consulta que conte os registros de uma tabela.
6. Registre no seu arquivo de estudo:
   - o nome do banco original;
   - o nome e o local do arquivo de backup;
   - o nome do banco restaurado;
   - o resultado da consulta de conferência.

> [!success] Resultado esperado
> Você terá uma cópia restaurada em um banco de teste e conseguirá provar, com uma consulta, que os dados foram recuperados.
