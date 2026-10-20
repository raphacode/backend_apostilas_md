# Gabarito-base

## 1:N
```sql
CREATE TABLE autores (
    id_autor BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL
);

CREATE TABLE livros (
    id_livro BIGSERIAL PRIMARY KEY,
    id_autor BIGINT NOT NULL REFERENCES autores(id_autor),
    titulo VARCHAR(255) NOT NULL
);
```

## N:N
```sql
CREATE TABLE alunos (
    id_aluno BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL
);

CREATE TABLE cursos (
    id_curso BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL
);

CREATE TABLE aluno_curso (
    id_aluno BIGINT NOT NULL REFERENCES alunos(id_aluno),
    id_curso BIGINT NOT NULL REFERENCES cursos(id_curso),
    PRIMARY KEY (id_aluno, id_curso)
);
```
