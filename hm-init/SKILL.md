---
name: hm-init
description: "Inicio de projeto world-class. Melhores ferramentas, melhor estrutura, melhores praticas desde o primeiro commit."
user_invocable: true
---

# /hm-init — Inicio de Projeto

Voce esta agora em **modo init**. Um projeto novo esta comecando. Seu trabalho e garantir que ele nasca certo.

## Principio central

O primeiro commit define o padrao. Um projeto world-class nao se torna world-class depois. Ele comeca world-class.

---

## Fase 1: Entendimento

Quando o fundador descrever o que quer construir, extraia:

1. **Tipo de produto:** SaaS, marketplace, API, ferramenta interna, mobile app, CLI, biblioteca
2. **Usuarios:** quem usa, volume esperado, contexto de uso
3. **Features core:** as 3-5 features que definem o MVP
4. **Integracoes:** pagamentos, email, auth social, APIs externas, webhooks
5. **Deploy target:** Vercel, AWS, GCP, self-hosted, mobile stores
6. **Restricoes:** timeline, time, budget, tech constraints

Se a descricao do fundador nao cobre os pontos criticos, faca UMA pergunta que cubra os gaps mais importantes. Nao faca 6 perguntas separadas.

---

## Fase 2: Decisoes tecnicas

Para cada decisao, avalie contra: adequacao ao projeto, ecossistema, performance, DX com Claude Code, e escalabilidade.

### Frontend

| Cenario | Recomendacao | Quando usar |
|---------|-------------|-------------|
| SaaS com landing + app | Next.js (App Router) | SSR/SSG nativo, ecossistema rico, deploy facil na Vercel |
| Dashboard/ERP complexo | Angular 19+ (standalone) | Forms complexos, tipagem forte, modularidade nativa, RxJS |
| App altamente interativo | React SPA (Vite) | Interatividade pura, sem necessidade de SSR |
| Site content-heavy | Astro | Performance maxima, islands architecture |
| App mobile | React Native / Expo | Codebase unico mobile, ecossistema maduro |

**Ecossistema de UI recomendado:**
- Next.js/React: shadcn/ui + Radix + Tailwind CSS
- Angular: Angular Material ou PrimeNG + Tailwind CSS
- Design system customizado: Tailwind CSS como foundation

### Backend

| Cenario | Recomendacao | Quando usar |
|---------|-------------|-------------|
| API REST rapida, SaaS | Node.js + Fastify (ou Express) + TypeScript | Performance, ecossistema npm, fullstack TS |
| API com ML/AI integrado | Python + FastAPI | Ecossistema de data science, async nativo, type hints |
| Enterprise, alta carga | Java 21+ + Spring Boot 3 | JVM performance, ecossistema enterprise maduro |
| Serverless/edge functions | Next.js API Routes ou Vercel Functions | Simplicidade, deploy integrado |
| Real-time/WebSocket | Node.js + Socket.io ou Fastify + ws | Event-driven nativo |

### Banco de dados

| Cenario | Recomendacao | Quando usar |
|---------|-------------|-------------|
| Relacional (maioria dos casos) | PostgreSQL | Sempre. E o padrao. |
| Schema flexivel necessario | MongoDB | Documentos heterogeneos, prototipacao rapida |
| Cache/sessoes/filas | Redis | Complementar ao banco principal |
| Fulltext search avancado | Elasticsearch / Meilisearch | Quando PostgreSQL FTS nao e suficiente |
| Vetor/embeddings | pgvector (PostgreSQL) ou Qdrant | AI/ML features |

### ORM / Query builder

| Stack | Recomendacao | Motivo |
|-------|-------------|--------|
| TypeScript + PostgreSQL | Drizzle ORM | Type-safe, leve, SQL-like, migrations integradas |
| TypeScript + PostgreSQL (alternativa) | Prisma | DX excelente, schema declarativo, mais overhead |
| Python + PostgreSQL | SQLAlchemy 2.0 + Alembic | Standard da industria, migrations robustas |
| Java + PostgreSQL | Spring Data JPA + Flyway | Integrado ao ecossistema Spring |

### Autenticacao

| Cenario | Recomendacao | Motivo |
|---------|-------------|--------|
| SaaS com social login | Supabase Auth ou Auth.js v5 | Setup rapido, providers integrados |
| Enterprise/SSO | Auth.js + adapter customizado | Flexibilidade para SAML/OIDC |
| API pura (B2B) | JWT com refresh tokens | Controle total, sem dependencia externa |
| Mobile app | Supabase Auth ou Firebase Auth | SDKs mobile nativos |

### Deploy

| Cenario | Recomendacao | Motivo |
|---------|-------------|--------|
| Next.js / frontend | Vercel | Zero config, preview deploys, edge |
| API/backend | Railway ou Fly.io | Simplicidade, containers, auto-scale |
| Full control | AWS (ECS/EKS) ou GCP (Cloud Run) | Enterprise, compliance, custom networking |
| Prototipo rapido | Docker Compose local | Sem custo, setup rapido |

Para cada escolha, escreva UMA frase de justificativa. Nao justifique o obvio.

---

## Fase 3: Scaffold

### Estrutura de pastas

A estrutura deve tornar a arquitetura visivel. Olhar as pastas deve dizer como o sistema funciona.

**Se Next.js (App Router):**
```
src/
  app/                  # rotas, layouts, loading/error states
    (auth)/             # grupo de rotas de auth
    (dashboard)/        # grupo de rotas do app
    api/                # API routes
  components/
    ui/                 # primitivos de UI (button, input, card)
    [feature]/          # componentes por feature
  lib/                  # utilitarios, configs, clients
  server/               # server actions, business logic
  types/                # tipos TypeScript compartilhados
```

**Se Angular 19+ (standalone):**
```
src/
  app/
    core/               # services singleton, guards, interceptors
    shared/             # componentes, pipes, directives reutilizaveis
    features/
      [feature]/        # components, services, routes por feature
    layouts/            # shell layouts
  environments/         # environment configs
```

**Se Python/FastAPI:**
```
src/
  api/                  # routers e endpoints
    v1/                 # versionamento de API
  core/                 # config, security, database, dependencies
  models/               # SQLAlchemy/ORM models
  schemas/              # Pydantic schemas (request/response)
  services/             # business logic
  utils/                # helpers compartilhados
tests/
  unit/
  integration/
migrations/             # Alembic migrations
```

**Se Java/Spring Boot:**
```
src/main/java/com/[org]/[project]/
  config/               # Spring configuration classes
  controller/           # REST controllers
  service/              # business logic
  repository/           # Spring Data repositories
  model/                # JPA entities
  dto/                  # request/response DTOs
  exception/            # custom exceptions + handler global
  security/             # auth config, filters, JWT
src/test/java/
```

### Configuracao de qualidade

Cheque cada item. Nao pule nenhum.

- [ ] TypeScript strict mode (se aplicavel): `strict: true` no `tsconfig.json`
- [ ] Linting configurado e opinado:
  - JS/TS: Biome (preferido — mais rapido, lint + format) ou ESLint + Prettier
  - Python: Ruff (preferido — mais rapido) ou Black + Flake8
  - Java: Checkstyle + SpotBugs
- [ ] Framework de testes instalado com teste exemplo RODANDO:
  - JS/TS: Vitest (preferido) ou Jest
  - Python: pytest
  - Java: JUnit 5 + Mockito
  - E2E: Playwright (se frontend)
- [ ] `.env.example` com TODAS as variaveis documentadas (descricao + valor de exemplo)
- [ ] `.gitignore` completo (node_modules, .env, dist, __pycache__, .class, etc.)
- [ ] Scripts uteis configurados no `package.json` / `Makefile` / `build.gradle`:
  - `dev` — rodar em desenvolvimento
  - `build` — build de producao
  - `test` — rodar testes
  - `lint` — rodar linter
  - `format` — rodar formatador (se separado do linter)

### Fundacao tecnica

Implemente o que o projeto precisa. Nao configure o que nao e necessario ainda.

- [ ] **Auth** (se o projeto precisa): login/register funcionando, protecao de rotas, token refresh
- [ ] **Schema do banco**: migration inicial com tabelas core, indexes, constraints
- [ ] **Endpoint de health check**: `GET /health` retornando status + versao
- [ ] **Error handling global**: middleware/handler que padroniza respostas de erro
- [ ] **Logger configurado**: logger estruturado (nao `console.log`), com niveis (info, warn, error)
- [ ] **CORS configurado**: restritivo por padrao, nao `*`
- [ ] **Environment config**: centralized, type-safe, com validacao (zod, pydantic, etc.)

---

## Fase 4: Verificacao

Antes de entregar, confirme TUDO. Nao entregue sem rodar.

### 1. Rodar o projeto
```bash
# Use o comando apropriado
npm run dev / python -m uvicorn src.main:app --reload / mvn spring-boot:run
```
O projeto roda sem erros? Se nao, corrija.

### 2. Rodar testes
```bash
npm test / pytest / mvn test
```
Os testes passam? Se nao, corrija.

### 3. Rodar linter
```bash
npm run lint / ruff check . / mvn checkstyle:check
```
Zero warnings, zero erros? Se nao, corrija.

### 4. Checklist final

- [ ] Projeto roda com UM comando
- [ ] Testes passam
- [ ] Lint passa
- [ ] Nenhum secret exposto (cheque com `Grep` para `password`, `secret`, `api_key`)
- [ ] Nenhum arquivo vazio ou placeholder
- [ ] `.env.example` completo
- [ ] README ou ARCHITECTURE.md com decisoes documentadas
- [ ] Todo arquivo que existe tem conteudo real e proposito claro

Se qualquer item falhar, corrija antes de entregar.

---

## Output

Apos o init, apresente:

```
DECISOES TECNICAS
| Escolha      | Ferramenta        | Justificativa                    |
|-------------|-------------------|----------------------------------|
| Frontend     | Next.js 15        | SSR nativo, deploy Vercel        |
| Backend      | API Routes        | Fullstack TS, simplicidade       |
| Banco        | PostgreSQL        | Relacional, pgvector futuro      |
| ORM          | Drizzle           | Type-safe, leve                  |
| Auth         | Auth.js v5        | Social login, session strategy   |
| Deploy       | Vercel            | Zero config, preview deploys     |
| Testes       | Vitest + Playwright| Unit + E2E                      |
| Lint/Format  | Biome             | Rapido, lint + format unificado  |

ESTRUTURA
[arvore de pastas com explicacao de cada diretorio principal]

VERIFICACAO
[x] Projeto roda: npm run dev -> http://localhost:3000
[x] Testes passam: 3/3
[x] Lint limpo: 0 warnings, 0 errors
[x] Nenhum placeholder

PROXIMO PASSO
[O que o fundador pode comecar a construir agora]
```

---

## O que AI erra em inits

- Instalar dependencias que o projeto ainda nao precisa ("pode ser util depois")
- Criar arquivos vazios como placeholder (`index.ts` que exporta nada, `utils.ts` vazio)
- Configurar CI/CD complexo quando o projeto nem tem features
- Usar versoes desatualizadas de frameworks — sempre cheque a versao mais recente
- Esquecer de configurar o `.env.example` com descricoes das variaveis
- Esquecer de adicionar scripts uteis no `package.json` (`dev`, `build`, `test`, `lint`)
- Scaffold de auth incompleto (login funciona, mas logout nao, ou refresh token falta)
- Instalar ESLint E Biome (escolha um — ambos no mesmo projeto causa conflitos)
- Instalar Prettier E Biome (Biome ja faz formatting)
- Nao rodar o projeto antes de entregar — "deve funcionar" nao e verificacao
- Configurar TypeScript sem `strict: true` — anula metade do beneficio
- Criar abstracoes prematuros (`BaseRepository`, `BaseService`) que ninguem precisa ainda
- README generico copiado de template sem informacao real do projeto

---

## Regras

- Nao pergunte "qual framework voce quer?" — recomende o melhor e explique por que
- Nao faca scaffold de arquivos vazios. Todo arquivo que existe tem conteudo real.
- Nao use pacotes deprecated ou sem manutencao
- Nao configure o que ainda nao e necessario. Escopo pro que o projeto precisa agora.
- Se a descricao do fundador for vaga, faca UMA pergunta de esclarecimento antes de prosseguir
- Sempre cheque a versao mais recente do framework antes de instalar. Use `npm info [pacote] version` ou equivalente.
- O projeto DEVE rodar apos o init. Rode e confirme. Se nao rodar, nao esta pronto.
- Toda dependencia precisa justificar sua existencia. "Pode ser que a gente precise depois" nao e justificativa.
- A estrutura de pastas precisa tornar a arquitetura visivel. Olhar as pastas deve dizer como o sistema funciona.
