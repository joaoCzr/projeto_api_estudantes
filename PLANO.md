# Plano: API Escola (FastAPI + SQLAlchemy + PostgreSQL)

Objetivo: refazer do zero, num repositório novo, uma API de **estudantes** e **matrículas**, só que mais completa que a versão de estudo: CRUD inteiro, relacionamento entre tabelas, validação, tratamento de erro, código organizado e testes.

**Tempo estimado:** 6 a 8 horas. Cada fase tem um checkpoint: só passe pra próxima quando ele funcionar.

---

## Regras do jogo (como me pedir ajuda)

Não vou te dar código pronto. Quando travar, me diga **em qual fase está, o que tentou e qual o erro** (cole o traceback inteiro). Eu respondo em níveis:

1. **Pergunta:** te devolvo uma pergunta que aponta pro caminho.
2. **Conceito:** te digo qual conceito ou parte da documentação estudar.
3. **Pista:** te digo *onde* está o problema (arquivo/linha), mas não *como* corrigir.
4. **Só se você pedir explicitamente "nível 4":** um exemplo mínimo de um caso *parecido*, nunca o seu código pronto.

Antes de me perguntar, tente: ler o erro de baixo pra cima, olhar o `/docs` (Swagger), e dar um `print()` no que você acha que está errado.

---

## Fase 0: Setup do projeto (~20 min)

**Tarefas**
- [ ] Criar o repositório novo e o ambiente virtual
- [ ] Copiar este `PLANO.md` e criar o `CLAUDE.md` na raiz do repositório (conteúdo logo abaixo), e commitar os dois
- [ ] Instalar as dependências: `fastapi`, `uvicorn`, `sqlalchemy`, `psycopg2-binary`, `pydantic`
- [ ] Gerar um `requirements.txt`
- [ ] Criar o `.gitignore`
- [ ] Criar o banco `escola` no PostgreSQL

**Conteúdo do `CLAUDE.md`**

O Claude Code lê esse arquivo sozinho ao abrir uma sessão no repositório. É ele que mantém o Claude no papel de mentor sem você ter que repetir as regras toda vez.

```markdown
# CLAUDE.md

## Contexto
Projeto de estudo: API Escola (FastAPI + SQLAlchemy + PostgreSQL), baseado no curso
de persistência de dados da Alura. O roteiro está em `PLANO.md`, dividido em fases
com checkpoints. Eu estou aprendendo; o objetivo é entender, não só terminar.

## Seu papel: mentor, não programador
- NÃO escreva, complete nem corrija código nos arquivos do projeto. Quem digita sou eu.
- NÃO me entregue o código pronto no chat, nem "só essa linha".
- Responda sempre em português.

## Como me ajudar (níveis, sempre comece pelo mais baixo)
1. **Pergunta:** devolva uma pergunta que aponte o caminho.
2. **Conceito:** diga qual conceito ou trecho da documentação estudar.
3. **Pista:** diga ONDE está o problema (arquivo/linha), mas não COMO corrigir.
4. **Exemplo:** só quando eu pedir "nível 4" explicitamente. Dê um exemplo mínimo
   de um caso PARECIDO, nunca a solução do meu código.

Só suba de nível se eu continuar travado depois de tentar.

## Antes de responder
- Pergunte em qual fase do `PLANO.md` eu estou, se não estiver claro.
- Se eu colar um erro, me ajude a LER o traceback (onde olhar, o que ele está
  dizendo) antes de apontar a causa.
- Você pode ler os arquivos e rodar comandos de leitura (testes, uvicorn, git status)
  pra entender o problema, mas não edite nada.

## Checkpoints
Quando eu disser que terminei uma fase, revise o checkpoint dela no `PLANO.md`:
me faça 1 ou 2 perguntas sobre o que eu fiz pra confirmar que entendi, e aponte
(sem corrigir) o que ficou faltando ou pode melhorar.

## Exceções
- Configuração de ambiente (instalar pacote, erro do PostgreSQL, venv, git) não é
  o foco do estudo: aí pode ser direto.
- Se eu disser "modo direto", pode explicar com código até eu dizer "volta pro modo mentor".
```

**Perguntas pra pensar**
- No repositório de estudo, os arquivos `.pyc` da pasta `__pycache__` foram commitados. Por que isso é ruim? O que mais deveria ir pro `.gitignore` além de `venv`?
- Qual comando gera o `requirements.txt` a partir do que está instalado?

**Checkpoint:** um `main.py` com uma rota `GET /` que retorna `{"status": "ok"}` rodando com o uvicorn, e você consegue abrir `http://localhost:8000/docs`.

---

## Fase 1: Conexão com o banco (~30 min)

**Tarefas**
- [ ] `database.py` com engine, sessão e Base
- [ ] Tirar a senha do banco de dentro do código (variável de ambiente ou arquivo `.env`)
- [ ] Função `get_db()` que abre e fecha a sessão

**Perguntas pra pensar**
- Por que o `get_db` usa `yield` em vez de `return`? O que acontece no `finally` se uma rota der erro no meio?
- Se a URL do banco estiver no código e você subir pro GitHub, qual o problema?
- `.env` deve ir pro git? E um `.env.example`?

**Conceitos pra pesquisar:** `os.getenv`, `python-dotenv`, "FastAPI dependencies with yield".

**Checkpoint:** a aplicação sobe sem erro de conexão. Teste com a senha errada de propósito e veja que erro aparece.

---

## Fase 2: Models (~30 min)

**Tarefas**
- [ ] Model `Estudante`: `id`, `nome` (obrigatório), `idade`, `email` (único)
- [ ] Model `Matricula`: `id`, `estudante_id` (FK), `nome_disciplina`
- [ ] Criar o **relacionamento** entre os dois (um estudante tem várias matrículas)
- [ ] Criar as tabelas ao iniciar a aplicação

**Perguntas pra pensar**
- Qual a diferença entre `ForeignKey` e `relationship`? Um existe no banco e o outro só no Python. Qual é qual?
- O que deveria acontecer com as matrículas quando um estudante é deletado? (pesquise `ondelete` e `cascade`)
- Como garantir que o mesmo estudante não se matricule duas vezes na mesma disciplina? (pesquise `UniqueConstraint`)

**Checkpoint:** abra o banco (pgAdmin, DBeaver ou `psql`) e confira que as duas tabelas existem com as colunas e a FK certas.

---

## Fase 3: Schemas Pydantic (~30 min)

**Tarefas**
- [ ] `EstudanteCreate`, `EstudanteUpdate`, `EstudanteResponse`
- [ ] `MatriculaCreate`, `MatriculaResponse`
- [ ] Um schema de estudante que inclui a lista de matrículas dele
- [ ] Validações: idade positiva, nome com tamanho mínimo, email válido

**Perguntas pra pensar**
- No projeto de estudo você usou `class Config: from_attributes = True`. Esse é o jeito do Pydantic v1. Como se escreve isso no v2? (pesquise `ConfigDict`)
- Por que o `EstudanteUpdate` precisa ter todos os campos opcionais?
- Por que separar o schema de entrada (`Create`) do de saída (`Response`)? Pense: o cliente manda o `id`?
- Você já viu no `minibanco/cliente.py` que o Pydantic converte `"30"` em `30`. Como você *restringe* um campo, em vez de só tipar? (pesquise `Field`, `EmailStr`)

**Checkpoint:** num script separado (igual ao `cliente.py`), crie schemas válidos e inválidos e veja os erros de validação.

---

## Fase 4: CRUD de Estudantes (~1h30)

| Método | Rota | O que faz | Status de sucesso |
|---|---|---|---|
| POST | `/estudantes/` | cria | 201 |
| GET | `/estudantes/` | lista | 200 |
| GET | `/estudantes/{id}` | busca um | 200 |
| PATCH | `/estudantes/{id}` | atualiza parcialmente | 200 |
| DELETE | `/estudantes/{id}` | remove | 204 |

**Tarefas**
- [ ] As 5 rotas acima
- [ ] Retornar **404** quando o estudante não existir
- [ ] Retornar **409** quando o email já estiver cadastrado
- [ ] Paginação na listagem (`skip` e `limit` como query params)

**Perguntas pra pensar**
- Como mudar o status code padrão de uma rota de 200 pra 201?
- No PATCH: se o cliente mandar só `{"idade": 20}`, como atualizar só a idade sem apagar o nome? (pesquise o parâmetro `exclude_unset` do `model_dump`)
- O que o banco faz se você inserir um email duplicado? Que exceção do SQLAlchemy aparece? Você deixa ela estourar como 500 ou trata?
- Depois de um erro no `commit`, o que precisa fazer com a sessão antes de usar de novo?

**Checkpoint:** pelo `/docs`, faça o ciclo inteiro: cria, lista, busca, atualiza, busca de novo, deleta, busca e recebe 404.

---

## Fase 5: Matrículas (~1h)

**Tarefas**
- [ ] `POST /estudantes/{id}/matriculas`: matricula o estudante numa disciplina
- [ ] `GET /estudantes/{id}/matriculas`: lista as matrículas de um estudante
- [ ] `DELETE /matriculas/{id}`: cancela uma matrícula
- [ ] `GET /estudantes/{id}` passa a devolver o estudante **com** as matrículas
- [ ] 404 se o estudante não existe, 409 se já está matriculado na disciplina

**Perguntas pra pensar**
- Por que a rota é `/estudantes/{id}/matriculas` e não `/matriculas` com `estudante_id` no corpo? As duas funcionam. Qual a vantagem de cada uma?
- Se o `estudante_id` não existir e você não checar antes, o que o PostgreSQL faz? E o que o usuário da API recebe?
- Para devolver as matrículas junto com o estudante, o que precisa mudar no `response_model`?

**Checkpoint:** cria um estudante, matricula em 3 disciplinas, tenta matricular de novo numa delas (409), busca o estudante e vê as 3. Deleta o estudante: o que aconteceu com as matrículas? Era o que você planejou na Fase 2?

---

## Fase 6: Organização do código (~45 min)

O `main.py` provavelmente está grande. Refatore:

```
app/
├── main.py          # cria o app e inclui os routers
├── database.py
├── models.py
├── schemas.py
├── crud.py          # funções que falam com o banco (sem nada de HTTP)
└── routers/
    ├── estudantes.py
    └── matriculas.py
```

**Perguntas pra pensar**
- Por que separar `crud.py` das rotas? A função do crud deveria lançar `HTTPException`? (Pense: e se um dia você chamar essa função de um script, sem HTTP?)
- Como usar `APIRouter` com `prefix` e `tags` pra organizar o `/docs`?
- Com a pasta `app/`, como muda o comando do uvicorn? E os imports?

**Checkpoint:** tudo que funcionava na Fase 5 continua funcionando, e o `/docs` mostra as rotas agrupadas por tag.

---

## Fase 7: Testes (~1h30)

**Tarefas**
- [ ] Instalar `pytest` e `httpx`
- [ ] Usar o `TestClient` do FastAPI
- [ ] Os testes rodam num **banco separado** (SQLite em memória ou um banco `escola_test`), nunca no banco de verdade
- [ ] No mínimo: criar estudante, buscar inexistente (404), email duplicado (409), PATCH parcial, matrícula duplicada (409), deletar

**Perguntas pra pensar**
- Como fazer a rota usar outro banco durante o teste sem mudar o código da rota? (pesquise `app.dependency_overrides`; é aqui que o `Depends(get_db)` mostra o valor dele)
- Como garantir que um teste não suja o banco do próximo? (pesquise `pytest fixture`)
- Se usar SQLite nos testes e PostgreSQL de verdade, que diferença pode te enganar?

**Checkpoint:** `pytest` passando com todos os testes verdes. Quebre uma rota de propósito e veja um teste falhar.

---

## Fase 8: Fechamento (~30 min)

- [ ] `README.md`: o que é o projeto, como instalar, como rodar, como testar, lista de rotas
- [ ] `.env.example` com as variáveis necessárias (sem a senha real)
- [ ] Rodar tudo do zero seguindo só o seu README (de preferência num clone limpo)
- [ ] Commits com mensagens que descrevem o que mudou (não só "continuando")

---

## Bônus (se sobrar tempo)

- [ ] Filtro na listagem: `GET /estudantes/?nome=jo`
- [ ] Model `Disciplina` de verdade, com a matrícula virando uma relação N:N entre estudante e disciplina
- [ ] Migrations com **Alembic** em vez de `create_all`
- [ ] Rodar o PostgreSQL com Docker (`docker-compose.yml`)

---

## Autoavaliação final

Responda sem olhar o código. Se travar em alguma, é o que vale revisar:

1. O que o `Depends` faz e por que ele ajuda nos testes?
2. Qual a diferença entre um model SQLAlchemy e um schema Pydantic?
3. O que `from_attributes` permite fazer?
4. Por que `db.refresh()` depois do `commit()`?
5. Quando usar 400, 404, 409 e 422? Quem gera o 422 sozinho?
6. O que `exclude_unset=True` muda num PATCH?
7. O que acontece com as matrículas quando você deleta um estudante, e por quê?
