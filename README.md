# Atividade Docker + CI — [SEU NOME]

> Preencha todos os campos marcados com `[...]` e substitua os prints de exemplo pelos seus. Salve as imagens em `docs/imagens/` e mantenha os nomes de arquivo indicados.

**Aluno(a):** Welder Barroso de Melo  
**Turma:** Vespertino  
**Data:** 24/07/2026 
**Aplicação usada:** docker/getting-started-app — To-Do em Node.js

**Sobre o app:**
- Runtime: Node.js 18+
- Início: `node src/index.js`
- Porta interna: `3000`
- Banco padrão: SQLite (`/etc/todos/todo.db`)
- Banco alternativo: MySQL, via variáveis `MYSQL_HOST`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DB`
- API: `GET /items`, `POST /items`

---

## 1. Como executar este projeto

```bash
git clone https://github.com/WelderBM/meu-projeto-docker
cd meu-projeto-docker
cp .env.example .env
docker compose up -d --build
```

**Acesse:** http://localhost:3000

**Para derrubar:**
- `docker compose down` (mantém dados)
- `docker compose down -v` (apaga dados)

---

## 2. Imagem e Dockerfile multi-stage

**Estágios utilizados:** builder (instala dependências) e estágio final (runtime enxuto)

**Imagem base:** [ex.: node:20-alpine]

**Usuário de execução:** node (não-root)

**Tamanho final da imagem:** [ex.: 180MB]

**Por que o multi-stage ajuda?** 

Dockerfile com dois estágios: `builder` (instala apenas dependências de produção com `npm ci --omit=dev`) e o estágio final, que copia seletivamente `node_modules`, `package.json` e `src` do builder — sem devDependencies e sem os arquivos de teste. Ao copiar só os artefatos necessários para o estágio final, a imagem não carrega ferramentas de build nem devDependencies, ficando menor e com menos superfície de ataque.

### Print 1 — build + docker images

![Build e docker images](docs/imagens/01-build-e-imagem.png)

### Print 2 — aplicação rodando com tarefas cadastradas

![App rodando com tarefas](docs/imagens/02-app-rodando.png)

---

## 3. Volumes e persistência

**Volume usado:** [nome] → montado em [caminho dentro do container]

### Print 3 — SEM volume: dados perdidos ao recriar o container

![Sem volume - dados perdidos](docs/imagens/03-sem-volume.png)

### Print 4 — COM volume: dados preservados

![Com volume - dados preservados](docs/imagens/04-com-volume.png)

**Exemplo de volumes nomeados:**

```
DRIVER    VOLUME NAME
local     f20f0604d8e9a224c5d574694a50d4c6e0b8b892cf3bf46ebb67e3f62c9f4c46
local     todo-db
```

**Diferença entre `docker compose down` e `docker compose down -v`:**

O primeiro remove containers e rede mas mantém os volumes nomeados (dados preservados); o segundo também remove os volumes, apagando os dados.

---

## 4. Rede

**Rede criada:** [nome]

**Serviços conectados:** app e db

**A porta do banco está exposta ao host?** Não — [justifique em 1 frase]

**Por que o app consegue chamar o host `mysql` / `db` sem saber o IP?**

Containers na mesma rede definida pelo usuário usam o DNS interno do Docker, que resolve o nome do container (ou `--network-alias`) para o IP correto automaticamente.

### Print 5 — docker network inspect

`docker network inspect todo-net`:

![Docker network inspect](docs/imagens/05-network-inspect.png)

### Print 6 — dados dentro do MySQL (select * from todo_items;)

`select * from todo_items;` dentro do MySQL:

![Select no MySQL](docs/imagens/06-select-mysql.png)

---

## 5. Docker Compose

**Serviços:** app, db

**Rede:** [nome]

**Volume:** [nome]

**Healthcheck em:** db

**depends_on com:** condition: service_healthy

**Variáveis sensíveis:** carregadas via `.env` (não versionado). Modelo em `.env.example`.

### Print 7 — docker compose ps

![docker compose ps com todos os serviços de pé](docs/imagens/07-compose-ps.png)

**Teste de persistência:**

![Tarefas mantidas após down sem -v](docs/imagens/08-persistencia-down.png)

![Lista vazia após down -v](docs/imagens/09-limpeza-down-v.png)

---

## 6. Integração Contínua (GitHub Actions)

**Arquivo do workflow:** `.github/workflows/ci.yml`

**Gatilhos:** push e pull_request

**O que o pipeline faz:**

1. [valida o compose]
2. [builda a imagem]
3. [sobe a stack]
4. [aguarda a app responder e testa criar uma tarefa via API]
5. [derruba a stack]

### Print 8 — execução verde ✅

![Execução verde do GitHub Actions](docs/imagens/10-ci-verde.png)

---

## 7. Quebra proposital do CI

**O que eu quebrei:** colocado caminho errado do index

**Erro que apareceu no log:** 

Error: Cannot find module '/app/src/indexx.js'

**Como o CI reagiu:** 

Na hora de "Esperar a aplicação responder" em actions deu bug, falhando no step de validação da aplicação.

**Como eu corrigi:** [o que foi alterado]

**Link do Pull Request:** [URL]

### Print 9 — execução vermelha ❌ + log do erro

![Execução vermelha do GitHub Actions](docs/imagens/11-ci-vermelho.png)

![Trecho do log com o erro](docs/imagens/12-ci-log-erro.png)

![Execução verde após a correção](docs/imagens/13-ci-corrigido.png)

---

## 8. Dificuldades e aprendizados

[3 a 5 linhas: o que travou, como resolveu, o que ficou mais claro sobre containers depois da atividade]

---

## 9. Checklist de autoavaliação

- [x] Dockerfile multi-stage funcionando
- [x] `.dockerignore` presente
- [x] Container não roda como root
- [x] Volume nomeado + persistência demonstrada
- [x] Rede nomeada + banco não exposto ao host
- [x] `compose.yaml` sobe tudo com um comando
- [x] `.env` no `.gitignore` e `.env.example` versionado
- [x] CI verde
- [x] PR com CI vermelho documentado
- [x] Todos os 9 prints no README
