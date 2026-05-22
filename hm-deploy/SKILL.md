---
name: hm-deploy
description: Validação de deploy por distribution model. Use antes de subir pra produção pela primeira vez, quando o ambiente local parou de funcionar, quando mudou infra, ou para validar que qualquer pessoa consegue subir o projeto do zero. Cobre 6 modelos distintos com checks próprios — Container/Docker, Serverless/Edge (Vercel/CF), Desktop (Electron), Mobile (Expo/RN), Library/SDK (npm/PyPI), CLI tool. Security Gate primeiro: se falha, não continua.
---

# /hm-deploy — Validação de Deploy (v3)

Você está agora em **modo deploy**. Seu trabalho e garantir que o projeto está pronto pra sair do local e ir pro mundo. Ou que o ambiente local esta saudavel e reprodutivel.

## Princípio central

Deploy não é o último passo. E uma camada de engenharia. Se o deploy e frágil, o produto e frágil. Se levantar o ambiente depende de conhecimento tribal, o projeto não está pronto. **Segurança de deploy não é checklist final — e pré-requisito.**

## Quando usar

- Antes de subir pra produção pela primeira vez
- Quando o ambiente local parou de funcionar
- Quando mudou infra (novo servico, nova porta, nova variavel)
- Pra validar que qualquer pessoa consegue subir o projeto do zero
- Depois de uma refatoracao significativa

## O que você valida

### 0. Distribution Model (PRIMEIRO — define os checks aplicaveis)

Identifique o modelo de distribuicao antes de qualquer auditoria. Cada modelo tem checks próprios — pular o que não se aplica e parte do trabalho.

| Modelo | Exemplos | Checks aplicaveis |
|---|---|---|
| **Container/Docker** | API com Postgres+Redis, monolitos | DOMÍNIO 1 inteiro, secrets em compose, multi-stage |
| **Serverless/Edge** | Vercel, Netlify, CF Workers | Cold start, env vars no dashboard, edge runtime, build output |
| **Desktop (Electron)** | macOS .app, Windows .exe | electron-builder config, contextIsolation, nodeIntegration false, code signing, secrets embedded warning, auto-update |
| **Mobile (Expo/RN)** | Apps na App Store / Play | EAS build, certs, App Transport Security, OTA updates, native modules ABI |
| **Library/SDK** | npm package, PyPI | semver, exports, types, lock file, supply chain, provenance, sem ANY no surface |
| **CLI tool** | Binario standalone | Cross-platform builds, signing, install path, autoupdate via release |

**Pula seções que não se aplicam.** Ex: app Electron NÃO tem `.dockerignore` — pule DOMÍNIO 1.1. Library NPM NÃO tem migrations — pule DOMÍNIO 3.

#### Checks por modelo (além do Security Gate)

**Container/Docker** (cobre DOMÍNIO 1 inteiro abaixo)

**Serverless/Edge:**
- Build output dentro do limite (Vercel: 50MB unzipped por function)?
- Cold start <1s pra rotas criticas? Se não, considerar warming ou edge runtime.
- Env vars sensíveis no dashboard, NUNCA em `next.config.js`?
- Edge runtime constraints respeitadas (sem `node:` modules, sem fs, sem better-sqlite3)?
- ISR/cache headers configurados?
- Domain + SSL configurados pré-deploy?

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
- Versão Electron com zero CVEs HIGH? `bun audit` / `npm audit` limpo?

**Mobile (Expo/RN):**
- EAS build profile (development, preview, production) configurado?
- Certificados/provisioning iOS via EAS?
- Android keystore versionado em local seguro (NÃO no repo)?
- `app.json` com bundle ID, version, build number corretos?
- Permissions declaradas (camera, location, etc) com justificativa?
- App Transport Security: zero `NSAllowsArbitraryLoads = true` em prod?
- OTA updates configuradas (`expo-updates`) ou versão nativa pinada?
- Native modules com versões compatíveis com SDK Expo?
- Firebase/analytics SDKs com config files certos por env?

**Library/SDK:**
- Lock file commitado (package-lock.json, poetry.lock)?
- `package.json` com `exports` map definido (ESM + CJS quando aplicavel)?
- Types em `.d.ts` ou `types` field correto?
- `files` field limita o que vai pro registry (sem `node_modules`, sem `.env`, sem testes)?
- Zero `any` no surface público?
- Semver respeitado: breaking change = major bump?
- README com installation + quick start + API reference?
- Provenance (npm provenance ou cosign) configurado pra supply chain?
- CI pública via OIDC (sem NPM_TOKEN secreto)?
- Tag git assinada pra cada release?

**CLI tool:**
- Cross-platform builds (Linux x64/arm64, macOS x64/arm64, Windows x64)?
- Signed binaries (Apple Developer ID, Authenticode)?
- Install path padroniza (Homebrew fórmula, Scoop manifest, .deb, AUR)?
- Help text (`--help`, `-h`) cobre todos os comandos?
- Versão reportada via `--version`?
- Update check opt-in (sem auto-update silencioso)?
- Logs vao pra path padrão do OS (XDG_STATE_HOME, ~/Library/Logs, %APPDATA%)?

### 1. Security Gate (Container — pular se modelo != Docker)

**Esta seção é bloqueante. Se qualquer item CRÍTICO falhar, o deploy NÃO está pronto. Não importa se tudo mais funciona.**

| Check | Criterio | Severidade |
|---|---|---|
| `.dockerignore` | Existe em CADA servico com Dockerfile. Exclui: `.env`, `.env.*`, `.git`, `node_modules`, `__pycache__`, `.venv`, `.next` | **CRÍTICO** — sem isso, secrets vazam nas layers da imagem Docker. Qualquer pessoa com acesso a imagem extrai .env |
| Dockerfile prod-ready | Multi-stage build. Sem gcc/dev-headers na imagem final. Sem `npm run dev`. Sem `--reload`. Sem `--debug`. | **CRÍTICO** — dev server em produção = hot reload instável + source maps expostos + info leak |
| Non-root user | Container roda como user não-root (`USER appuser`) | **ALTO** — se o container for comprometido, atacante tem root |
| Build secrets | Nenhum secret em Dockerfile (COPY, ARG, ENV com valores reais) | **CRÍTICO** — visível em `docker history`, irrecuperavel |
| Secrets em compose | docker-compose.yml usa `${VAR}` ou `env_file`, nunca valores literais de secrets | **CRÍTICO** — compose commitado no git = secrets públicos |
| Entrypoint separado | dev (com --reload) e prod (sem --reload) sao entrypoints diferentes | **ALTO** — um único entrypoint tenta servir dois propositos e falha em ambos |
| Dependency audit | Zero CVEs HIGH/CRITICAL em dependências (`npm audit`, `pip audit`) | **ALTO** — vulnerabilidade conhecida e porta aberta |
| CORS | Configurável via env var. Nunca `*` em produção. Nunca hardcoded localhost. | **ALTO** — CORS `*` permite qualquer origem fazer requests autenticados |
| Swagger/Debug | `/docs`, `/redoc`, debug mode desabilitados quando `APP_ENV != development` | **ALTO** — endpoints de documentação expoe toda a API surface |

**Se `.dockerignore` não existe: PARA TUDO. Cria antes de continuar a validação.**

### 1. Docker & Containers

**Subida:**
- `docker compose up` sobe todos os servicos sem erro?
- Todos os containers ficam healthy? (não só running — healthy)
- A ordem de dependência esta correta? (banco antes da API, etc)
- Logs dos containers mostram startup limpo?

**Rebuild:**
- Mudancas de código sao refletidas apos `docker compose build <service> && docker compose up -d <service>`?
- O Dockerfile usa multi-stage build?
- Cache de layers esta otimizado? (deps antes de code copy)
- Imagem final não tem ferramentas de dev desnecessarias?
- Tamanho da imagem final e razoável? (Python slim < 200MB, Node alpine < 150MB)

**Dados sagrados:**
- Volumes sao nomeados (nunca anonymous)?
- `docker compose down` (sem -v) preserva todos os dados?
- Dados do banco sobrevivem a rebuild de container?
- Se tem dados de produção local, estao protegidos contra `down -v`?

### 2. Environment & Configuração

- `.env.example` existe e tem TODAS as variaveis necessarias?
- Nenhum secret esta hardcoded no código ou no docker-compose.yml?
- Variaveis sensíveis estao marcadas como tal no .env.example? (com `change-me` ou `your-key-here`)
- Valores padrão fazem sentido pra dev local?
- Ports estao documentados e não colidem com outros projetos?

**Checklist de ports:**

Manter um registro vivo dos ports usados por todos os projetos do mesmo ecossistema que rodam em paralelo na máquina do dev. Cada projeto novo deve consultar e reservar ports que não colidem.

Estrutura típica de port allocation:

| Projeto | API/Backend | Web/Frontend | Postgres | Redis | Outros (MinIO/S3, etc) |
|---|---|---|---|---|---|
| <proj-1> | 8000 | 3000 | 5432 | 6379 | — |
| <proj-2> | 8001 | 3001 | 5433 | 6380 | — |
| <proj-3> | 8002 | 3002 | 5434 | 6381 | MinIO 9000-9001 |

Faixa sugerida pra novo projeto: API 8000+N, Web 3000+N, Postgres 5432+N, Redis 6379+N — onde N e o próximo livre.

**Anti-patterns:**
- Dois projetos disputando porta 5432 ou 3000.
- Ports hardcoded em código (deve ser env var).
- Ports diferentes em `.env.example` vs `docker-compose.yml`.

### 3. Database & Migrations

- Migrations rodam automaticamente no boot do container?
- Migrations estao em ordem e não tem gaps?
- Nenhuma migration e destrutiva sem ser reversível?
- Schema atual reflete todas as migrations aplicadas?
- Conexão do app com o banco funciona logo apos subir?

### 4. Health & Monitoramento

- Endpoint de health check existe? (`/health` ou `/api/health`)
- Health check retorna status dos servicos dependentes (banco, cache, etc)?
- Health check NÃO retorna só `{"status": "ok"}` — verifica conexão real com DB e Redis
- Logs sao estruturados e úteis (não verbose demais)?
- Erros sao logados com contexto suficiente pra debuggar?

### 5. Reprodutibilidade

O teste definitivo: **clone limpo**.
1. Clone o repo
2. Copie `.env.example` pra `.env`
3. Rode `docker compose up`
4. O projeto funciona?

Se qualquer passo extra é necessário, esta faltando documentação ou automação.

### 6. Segurança de deploy (checklist complementar)

Além do Security Gate (seção 0), verificar:
- Nenhum port desnecessario exposto
- HTTPS configurado (se produção/homologacao)
- Secrets não estao nos logs de build
- `.env` esta no `.gitignore`
- Rate limiting em endpoints públicos
- Security headers configurados (X-Content-Type-Options, X-Frame-Options, etc.)
- Logs não contem secrets, tokens, ou senhas

### 7. Scripts & DX

- Existe um README ou ARCHITECTURE.md com instrucoes de setup?
- O setup é um comando (ou no máximo dois)?
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
Conexão: OK/falhou
Dados persistentes: sim/não

HEALTH
Endpoint: existe/não existe
Servicos: todos healthy/X unhealthy

REPRODUTIBILIDADE
Clone limpo: funciona/falha no passo X

SEGURANÇA COMPLEMENTAR
[Check]: OK/issue (detalhes)

VEREDICTO
Pronto pra deploy / BLOQUEADO — X criticos, Y altos pra resolver primeiro
```

> Para auditoria de segurança completa (OWASP ASVS, LLM, multi-tenant, business logic), usar `/hm-security` apos o deploy estar funcional.

## Regras
- **Security Gate é a PRIMEIRA coisa que roda. Se falha, não continua.**
- Nunca assuma que "funciona na minha máquina" e suficiente
- Dados sao sagrados. Se a validação mostra risco de perda de dados, é CRÍTICO.
- Todo finding tem fix especifico. Não só "configure melhor."
- Se o projeto não sobe do zero com um comando, é finding.
- Se um secret está exposto, é CRÍTICO. Sem exceção.
- **Se `.dockerignore` não existe, é CRÍTICO. Ponto.**
- **Se Dockerfile roda dev server ou --reload, é CRÍTICO. Ponto.**
- **Se container roda como root, é ALTO. Ponto.**
- Teste o clone limpo mentalmente (ou de fato). Cada passo manual e divida técnica.
- O padrão: um engenheiro novo entra no time na segunda-feira e tem o projeto rodando antes do almoco.
