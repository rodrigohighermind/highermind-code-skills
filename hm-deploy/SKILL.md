# /hm-deploy — Validacao de Deploy (v2)

Voce esta agora em **modo deploy**. Seu trabalho e garantir que o projeto esta pronto pra sair do local e ir pro mundo. Ou que o ambiente local esta saudavel e reprodutivel.

## Principio central

Deploy nao e o ultimo passo. E uma camada de engenharia. Se o deploy e fragil, o produto e fragil. Se levantar o ambiente depende de conhecimento tribal, o projeto nao esta pronto. **Seguranca de deploy nao e checklist final — e pre-requisito.**

## Quando usar

- Antes de subir pra producao pela primeira vez
- Quando o ambiente local parou de funcionar
- Quando mudou infra (novo servico, nova porta, nova variavel)
- Pra validar que qualquer pessoa consegue subir o projeto do zero
- Depois de uma refatoracao significativa

## O que voce valida

### 0. Security Gate (PRIMEIRO — antes de tudo)

**Esta secao e bloqueante. Se qualquer item CRITICO falhar, o deploy NAO esta pronto. Nao importa se tudo mais funciona.**

| Check | Criterio | Severidade |
|---|---|---|
| `.dockerignore` | Existe em CADA servico com Dockerfile. Exclui: `.env`, `.env.*`, `.git`, `node_modules`, `__pycache__`, `.venv`, `.next` | **CRITICO** — sem isso, secrets vazam nas layers da imagem Docker. Qualquer pessoa com acesso a imagem extrai .env |
| Dockerfile prod-ready | Multi-stage build. Sem gcc/dev-headers na imagem final. Sem `npm run dev`. Sem `--reload`. Sem `--debug`. | **CRITICO** — dev server em producao = hot reload instavel + source maps expostos + info leak |
| Non-root user | Container roda como user nao-root (`USER appuser`) | **ALTO** — se o container for comprometido, atacante tem root |
| Build secrets | Nenhum secret em Dockerfile (COPY, ARG, ENV com valores reais) | **CRITICO** — visivel em `docker history`, irrecuperavel |
| Secrets em compose | docker-compose.yml usa `${VAR}` ou `env_file`, nunca valores literais de secrets | **CRITICO** — compose commitado no git = secrets publicos |
| Entrypoint separado | dev (com --reload) e prod (sem --reload) sao entrypoints diferentes | **ALTO** — um unico entrypoint tenta servir dois propositos e falha em ambos |
| Dependency audit | Zero CVEs HIGH/CRITICAL em dependencias (`npm audit`, `pip audit`) | **ALTO** — vulnerabilidade conhecida e porta aberta |
| CORS | Configuravel via env var. Nunca `*` em producao. Nunca hardcoded localhost. | **ALTO** — CORS `*` permite qualquer origem fazer requests autenticados |
| Swagger/Debug | `/docs`, `/redoc`, debug mode desabilitados quando `APP_ENV != development` | **ALTO** — endpoints de documentacao expoe toda a API surface |

**Se `.dockerignore` nao existe: PARA TUDO. Cria antes de continuar a validacao.**

### 1. Docker & Containers

**Subida:**
- `docker compose up` sobe todos os servicos sem erro?
- Todos os containers ficam healthy? (nao so running — healthy)
- A ordem de dependencia esta correta? (banco antes da API, etc)
- Logs dos containers mostram startup limpo?

**Rebuild:**
- Mudancas de codigo sao refletidas apos `docker compose build <service> && docker compose up -d <service>`?
- O Dockerfile usa multi-stage build?
- Cache de layers esta otimizado? (deps antes de code copy)
- Imagem final nao tem ferramentas de dev desnecessarias?
- Tamanho da imagem final e razoavel? (Python slim < 200MB, Node alpine < 150MB)

**Dados sagrados:**
- Volumes sao nomeados (nunca anonymous)?
- `docker compose down` (sem -v) preserva todos os dados?
- Dados do banco sobrevivem a rebuild de container?
- Se tem dados de producao local, estao protegidos contra `down -v`?

### 2. Environment & Configuracao

- `.env.example` existe e tem TODAS as variaveis necessarias?
- Nenhum secret esta hardcoded no codigo ou no docker-compose.yml?
- Variaveis sensiveis estao marcadas como tal no .env.example? (com `change-me` ou `your-key-here`)
- Valores padrao fazem sentido pra dev local?
- Ports estao documentados e nao colidem com outros projetos?

**Checklist de ports (projetos Higher Mind):**
| Projeto | API | Web | Postgres | Redis | Outros |
|---|---|---|---|---|---|
| higher-mind-os | 8000 | 3000 | 5432 | 6379 | — |
| scout | 8001 | 3000 | 5433 | 6381 | — |
| hm-finance | 8003 | 3001 | 5434 | — | — |
| orion-finance | 8005 | 3002 | 5436 | 6383 | MinIO 9000-9001 |
| hm-product-lab | 8004 | — | 5435 | 6382 | — |

Se o projeto e novo, escolher ports que nao colidem.

### 3. Database & Migrations

- Migrations rodam automaticamente no boot do container?
- Migrations estao em ordem e nao tem gaps?
- Nenhuma migration e destrutiva sem ser reversivel?
- Schema atual reflete todas as migrations aplicadas?
- Conexao do app com o banco funciona logo apos subir?

### 4. Health & Monitoramento

- Endpoint de health check existe? (`/health` ou `/api/health`)
- Health check retorna status dos servicos dependentes (banco, cache, etc)?
- Health check NAO retorna so `{"status": "ok"}` — verifica conexao real com DB e Redis
- Logs sao estruturados e uteis (nao verbose demais)?
- Erros sao logados com contexto suficiente pra debuggar?

### 5. Reprodutibilidade

O teste definitivo: **clone limpo**.
1. Clone o repo
2. Copie `.env.example` pra `.env`
3. Rode `docker compose up`
4. O projeto funciona?

Se qualquer passo extra e necessario, esta faltando documentacao ou automacao.

### 6. Seguranca de deploy (checklist complementar)

Alem do Security Gate (secao 0), verificar:
- Nenhum port desnecessario exposto
- HTTPS configurado (se producao/homologacao)
- Secrets nao estao nos logs de build
- `.env` esta no `.gitignore`
- Rate limiting em endpoints publicos
- Security headers configurados (X-Content-Type-Options, X-Frame-Options, etc.)
- Logs nao contem secrets, tokens, ou senhas

### 7. Scripts & DX

- Existe um README ou ARCHITECTURE.md com instrucoes de setup?
- O setup e um comando (ou no maximo dois)?
- Scripts de desenvolvimento estao documentados? (como rodar testes, como rebuildar, etc)
- Makefile ou scripts de conveniencia existem se necessario?

## Formato do output

```
SECURITY GATE
[Check]: PASSED/FAILED (detalhes)
Gate: PASSED / BLOCKED (N criticos, M altos)

CONTAINERS
[Servico]: healthy/unhealthy (detalhes)
Build: OK/FALHOU (detalhes)
Dados: protegidos/em risco (detalhes)

ENVIRONMENT
.env.example: completo/incompleto (variaveis faltando)
Secrets: seguros/expostos (detalhes)
Ports: OK/conflito (detalhes)

DATABASE
Migrations: OK/falhou (detalhes)
Conexao: OK/falhou
Dados persistentes: sim/nao

HEALTH
Endpoint: existe/nao existe
Servicos: todos healthy/X unhealthy

REPRODUTIBILIDADE
Clone limpo: funciona/falha no passo X

SEGURANCA COMPLEMENTAR
[Check]: OK/issue (detalhes)

VEREDICTO
Pronto pra deploy / BLOQUEADO — X criticos, Y altos pra resolver primeiro
```

## Regras
- **Security Gate e a PRIMEIRA coisa que roda. Se falha, nao continua.**
- Nunca assuma que "funciona na minha maquina" e suficiente
- Dados sao sagrados. Se a validacao mostra risco de perda de dados, e CRITICO.
- Todo finding tem fix especifico. Nao so "configure melhor."
- Se o projeto nao sobe do zero com um comando, e finding.
- Se um secret esta exposto, e CRITICO. Sem excecao.
- **Se `.dockerignore` nao existe, e CRITICO. Ponto.**
- **Se Dockerfile roda dev server ou --reload, e CRITICO. Ponto.**
- **Se container roda como root, e ALTO. Ponto.**
- Teste o clone limpo mentalmente (ou de fato). Cada passo manual e divida tecnica.
- O padrao: um engenheiro novo entra no time na segunda-feira e tem o projeto rodando antes do almoco.
