# /hm-deploy — Validação de Deploy

Você está agora em **modo deploy**. Seu trabalho é garantir que o projeto está pronto pra sair do local e ir pro mundo. Ou que o ambiente local está saudável e reprodutível.

## Princípio central

Deploy não é o último passo. É uma camada de engenharia. Se o deploy é frágil, o produto é frágil. Se levantar o ambiente depende de conhecimento tribal, o projeto não está pronto.

## Quando usar

- Antes de subir pra produção pela primeira vez
- Quando o ambiente local parou de funcionar
- Quando mudou infra (novo serviço, nova porta, nova variável)
- Pra validar que qualquer pessoa consegue subir o projeto do zero
- Depois de uma refatoração significativa

## O que você valida

### 1. Docker & Containers

**Subida:**
- `docker compose up` sobe todos os serviços sem erro?
- Todos os containers ficam healthy? (não só running — healthy)
- A ordem de dependência está correta? (banco antes da API, etc)
- Logs dos containers mostram startup limpo?

**Rebuild:**
- Mudanças de código são refletidas após `docker compose build <service> && docker compose up -d <service>`?
- O Dockerfile usa multi-stage build quando apropriado?
- Cache de layers está otimizado? (deps antes de code copy)
- Imagem final não tem ferramentas de dev desnecessárias?

**Dados sagrados:**
- Volumes são nomeados (nunca anonymous)?
- `docker compose down` (sem -v) preserva todos os dados?
- Dados do banco sobrevivem a rebuild de container?
- Se tem dados de produção local, estão protegidos contra `down -v`?

### 2. Environment & Configuração

- `.env.example` existe e tem TODAS as variáveis necessárias?
- Nenhum secret está hardcoded no código ou no docker-compose.yml?
- Variáveis sensíveis estão marcadas como tal no .env.example?
- Valores padrão fazem sentido pra dev local?
- Ports estão documentados e não colidem com outros projetos?

**Checklist de ports (projetos Higher Mind):**
| Projeto | API | Web | Postgres | Redis |
|---|---|---|---|---|
| higher-mind-os | 8000 | 3000 | 5432 | 6379 |
| scout | 8001 | 3000 | 5433 | 6381 |
| hm-finance | 8003 | 3001 | 5434 | — |

Se o projeto é novo, escolher ports que não colidem.

### 3. Database & Migrations

- Migrations rodam automaticamente no boot do container?
- Migrations estão em ordem e não têm gaps?
- Nenhuma migration é destrutiva sem ser reversível?
- Schema atual reflete todas as migrations aplicadas?
- Conexão do app com o banco funciona logo após subir?

### 4. Health & Monitoramento

- Endpoint de health check existe? (`/health` ou `/api/health`)
- Health check retorna status dos serviços dependentes (banco, cache, etc)?
- Logs são estruturados e úteis (não verbose demais)?
- Erros são logados com contexto suficiente pra debuggar?

### 5. Reprodutibilidade

O teste definitivo: **clone limpo**.
1. Clone o repo
2. Copie `.env.example` pra `.env`
3. Rode `docker compose up`
4. O projeto funciona?

Se qualquer passo extra é necessário, está faltando documentação ou automação.

### 6. Segurança de deploy

- Nenhum port desnecessário exposto
- Nenhum serviço de debug ativo em configuração de produção
- CORS configurado corretamente (não `*` em produção)
- HTTPS configurado (se produção)
- Secrets não estão nos logs de build
- `.env` está no `.gitignore`
- `.dockerignore` exclui node_modules, .env, .git, etc

### 7. Scripts & DX

- Existe um README ou ARCHITECTURE.md com instruções de setup?
- O setup é um comando (ou no máximo dois)?
- Scripts de desenvolvimento estão documentados? (como rodar testes, como rebuildar, etc)
- Makefile ou scripts de conveniência existem se necessário?

## Formato do output

```
CONTAINERS
[Serviço]: healthy/unhealthy (detalhes)
Build: OK/FALHOU (detalhes)
Dados: protegidos/em risco (detalhes)

ENVIRONMENT
.env.example: completo/incompleto (variáveis faltando)
Secrets: seguros/expostos (detalhes)
Ports: OK/conflito (detalhes)

DATABASE
Migrations: OK/falhou (detalhes)
Conexão: OK/falhou
Dados persistentes: sim/não

HEALTH
Endpoint: existe/não existe
Serviços: todos healthy/X unhealthy

REPRODUTIBILIDADE
Clone limpo: funciona/falha no passo X

SEGURANÇA
[Check]: OK/issue (detalhes)

VEREDICTO
Pronto pra deploy / X issues pra resolver primeiro
```

## Regras
- Nunca assuma que "funciona na minha máquina" é suficiente
- Dados são sagrados. Se a validação mostra risco de perda de dados, é CRÍTICO.
- Todo finding tem fix específico. Não só "configure melhor."
- Se o projeto não sobe do zero com um comando, é finding.
- Se um secret está exposto, é CRÍTICO. Sem exceção.
- Teste o clone limpo mentalmente (ou de fato). Cada passo manual é dívida técnica.
- O padrão: um engenheiro novo entra no time na segunda-feira e tem o projeto rodando antes do almoço.
