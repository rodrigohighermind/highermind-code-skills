# /hm-deploy — Validacao de Deploy (v3)

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

### 0. Distribution Model (PRIMEIRO — define os checks aplicaveis)

Identifique o modelo de distribuicao antes de qualquer auditoria. Cada modelo tem checks proprios — pular o que nao se aplica e parte do trabalho.

| Modelo | Exemplos | Checks aplicaveis |
|---|---|---|
| **Container/Docker** | API com Postgres+Redis, monolitos | Dominio 1 inteiro, secrets em compose, multi-stage |
| **Serverless/Edge** | Vercel, Netlify, CF Workers | Cold start, env vars no dashboard, edge runtime, build output |
| **Desktop (Electron)** | macOS .app, Windows .exe | electron-builder config, contextIsolation, nodeIntegration false, code signing, secrets embedded warning, auto-update |
| **Mobile (Expo/RN)** | Apps na App Store / Play | EAS build, certs, App Transport Security, OTA updates, native modules ABI |
| **Library/SDK** | npm package, PyPI | semver, exports, types, lock file, supply chain, provenance, sem ANY no surface |
| **CLI tool** | Binario standalone | Cross-platform builds, signing, install path, autoupdate via release |

**Pula secoes que nao se aplicam.** Ex: app Electron NAO tem `.dockerignore` — pule Dominio 1.1. Library NPM NAO tem migrations — pule Dominio 3.

#### Checks por modelo (alem do Security Gate)

**Container/Docker** (cobre Dominio 1 inteiro abaixo)

**Serverless/Edge:**
- Build output dentro do limite (Vercel: 50MB unzipped por function)?
- Cold start <1s pra rotas criticas? Se nao, considerar warming ou edge runtime.
- Env vars sensiveis no dashboard, NUNCA em `next.config.js`?
- Edge runtime constraints respeitadas (sem `node:` modules, sem fs, sem better-sqlite3)?
- ISR/cache headers configurados?
- Domain + SSL configurados pre-deploy?

**Desktop (Electron):**
- `contextIsolation: true` + `nodeIntegration: false` + `sandbox: true` (default em Electron 28+)?
- Sem preload script expondo APIs perigosas? Se tem preload, exporta apenas APIs minimas via `contextBridge`?
- `webSecurity: true` (NUNCA `false`)?
- Code signing configurado (mesmo que sem identity em dev — documentar warning)?
- Auto-update via electron-updater? Ou ship manual?
- **Secrets bundled na .app**: `.env.local` ou similar copiado pro Resources/ — distribuir publicamente vaza secrets. Documentar.
- Native modules (better-sqlite3, sharp) com ABI Electron rebuild correto?
- `app.disableHardwareAcceleration()` se necessario?
- `setAppUserModelId` (Windows) e bundle id (macOS) corretos?
- Versao Electron com zero CVEs HIGH? `bun audit` / `npm audit` limpo?

**Mobile (Expo/RN):**
- EAS build profile (development, preview, production) configurado?
- Certificados/provisioning iOS via EAS?
- Android keystore versionado em local seguro (NAO no repo)?
- `app.json` com bundle ID, version, build number corretos?
- Permissions declaradas (camera, location, etc) com justificativa?
- App Transport Security: zero `NSAllowsArbitraryLoads = true` em prod?
- OTA updates configuradas (`expo-updates`) ou versao nativa pinada?
- Native modules com versoes compativeis com SDK Expo?
- Firebase/analytics SDKs com config files certos por env?

**Library/SDK:**
- Lock file commitado (package-lock.json, poetry.lock)?
- `package.json` com `exports` map definido (ESM + CJS quando aplicavel)?
- Types em `.d.ts` ou `types` field correto?
- `files` field limita o que vai pro registry (sem `node_modules`, sem `.env`, sem testes)?
- Zero `any` no surface publico?
- Semver respeitado: breaking change = major bump?
- README com installation + quick start + API reference?
- Provenance (npm provenance ou cosign) configurado pra supply chain?
- CI publica via OIDC (sem NPM_TOKEN secreto)?
- Tag git assinada pra cada release?

**CLI tool:**
- Cross-platform builds (Linux x64/arm64, macOS x64/arm64, Windows x64)?
- Signed binaries (Apple Developer ID, Authenticode)?
- Install path padroniza (Homebrew formula, Scoop manifest, .deb, AUR)?
- Help text (`--help`, `-h`) cobre todos os comandos?
- Versao reportada via `--version`?
- Update check opt-in (sem auto-update silencioso)?
- Logs vao pra path padrao do OS (XDG_STATE_HOME, ~/Library/Logs, %APPDATA%)?

### 1. Security Gate (Container — pular se modelo != Docker)

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

**Checklist de ports:**

Manter um registro vivo dos ports usados por todos os projetos do mesmo ecossistema que rodam em paralelo na maquina do dev. Cada projeto novo deve consultar e reservar ports que nao colidem.

Estrutura tipica de port allocation:

| Projeto | API/Backend | Web/Frontend | Postgres | Redis | Outros (MinIO/S3, etc) |
|---|---|---|---|---|---|
| <proj-1> | 8000 | 3000 | 5432 | 6379 | — |
| <proj-2> | 8001 | 3001 | 5433 | 6380 | — |
| <proj-3> | 8002 | 3002 | 5434 | 6381 | MinIO 9000-9001 |

Faixa sugerida pra novo projeto: API 8000+N, Web 3000+N, Postgres 5432+N, Redis 6379+N — onde N e o proximo livre.

**Anti-patterns:**
- Dois projetos disputando porta 5432 ou 3000.
- Ports hardcoded em codigo (deve ser env var).
- Ports diferentes em `.env.example` vs `docker-compose.yml`.

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

> Para auditoria de seguranca completa (OWASP ASVS, LLM, multi-tenant, business logic), usar `/hm-security` apos o deploy estar funcional.

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
