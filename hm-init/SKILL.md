---
name: hm-init
description: Início de projeto novo no padrão Higher Mind. Use quando vai iniciar um repositório do zero — escolhe a melhor stack, monta estrutura, infra local Docker-first, segurança day-1 (.dockerignore, multi-stage, non-root), Memory + CLAUDE.md cravados, configs obrigatórias por stack (Next.js, Supabase, Tailwind v4, Tauri/Electron). Para CLI especificamente, complementa com /hm-cli.
---

# /hm-init — Início de Projeto (v3)

Você está agora em **modo init**. Um projeto novo esta comecando. Seu trabalho e garantir que ele nasca certo.

> **Antes de iniciar**, considere rodar `/hm-align` pra validar que isso é a coisa certa pra construir agora, e `/hm-sequoia` pra checar se aponta pro futuro (Autopilot/Outcome) e não pro passado (Copilot/Ferramenta). Init sem alinhamento gera código bonito que ninguém usa.
>
> **Se o projeto e CLI especificamente** (terminal app, ferramenta de linha de comando), use `/hm-cli` em vez desta — stack obrigatoria (Bun + Ink + bun:sqlite), filosofia agentic-first e patterns visuais cravados.

## Princípio central

O primeiro commit define o padrão. Um projeto world-class não se torna world-class depois. Ele comeca world-class. **Segurança não é fase. E fundacao.**

## Seu trabalho

Quando o fundador descrever o que quer construir, você:

### 1. Avalia e escolhe a melhor stack

Não a popular. Não a padrão. A melhor pra ESSE projeto especifico. Use o framework de decisão:

| Criterio | Peso | Pergunta |
|---|---|---|
| **Fit pro problema** | CRÍTICO | Essa ferramenta resolve o core do problema melhor que as alternativas? |
| **Performance** | ALTO | Latencia, throughput, cold starts — atende os requisitos do projeto? |
| **Custo em produção** | ALTO | API calls, hosting, bandwidth — quanto custa rodar isso em escala? |
| **Segurança** | ALTO | Histórico de CVEs, atualizacoes de segurança, supply chain confiavel? |
| **Maturidade** | MEDIO | Tem docs, comunidade, edge cases resolvidos? Ou e bleeding-edge com armadilhas? |
| **Ecossistema** | MEDIO | Libs, integracoes, tooling — o ecossistema resolve ou você vai ter que reinventar? |
| **DX (Developer Experience)** | MEDIO | Velocidade de iteração, debugging, deploy — o dia a dia e fluido? |
| **Hiring pool** | BAIXO* | Se o projeto vai ter time, tem gente que sabe isso? |

*Hiring pool e baixo porque projetos Higher Mind sao builder-first. Mas conta se vai escalar time.

**Justifique cada escolha em uma frase.** Se duas opções sao próximas, explique por que uma vence.

**Anti-patterns de escolha:**
- "Todo mundo usa" não é razão
- "E o que eu conheco" não é razão (a menos que o deadline justifique)
- "Pode ser que a gente precise" não é razão pra adicionar dependência

### 2. Define a arquitetura como agent-first (quando aplicavel)

Se o projeto tem componente de AI/agente:
- **Agent-first**: o agente executa, a UI da visibilidade e override
- **Chat-first**: a interface principal e conversa, não formularios
- Antes de criar UI pra algo, pergunte: "O agente consegue fazer isso via conversa?"
- Dashboards existem pra visibilidade, não pra input principal

Se o projeto e puramente frontend/ferramenta, ignore este passo.

### 3. Monta a estrutura

- Estrutura de pastas que torna a arquitetura visível
- Boundaries entre módulos claros e respeitados
- Convencoes de naming consistentes
- Um engenheiro senior entenderia o projeto em 10 minutos

### 4. Configura qualidade desde o dia um

- TypeScript strict mode (se aplicavel) / type hints completos (Python)
- Linting e formatacao com config opinada
- Framework de testes pronto pra usar
- Gerenciamento de environments (.env com .env.example)
- Git hooks se apropriado

### 5. Monta a infraestrutura local

- **Docker Compose** como padrão pra local dev (banco, cache, servicos)
- Ports documentados e não conflitantes com outros projetos
- Volumes nomeados (dados sao sagrados — nunca perder dados)
- Health checks nos servicos
- Scripts de setup (um comando pra subir tudo)
- Migrations automáticas no boot

### 6. Segurança desde o primeiro commit

**Esta seção e OBRIGATORIA. Nenhum projeto nasce sem isso.**

#### .dockerignore (OBRIGATÓRIO em todo projeto com Docker)
Criar `.dockerignore` no root de CADA servico que tem Dockerfile:
```
.git
.gitignore
.env
.env.*
!.env.example
node_modules
__pycache__
*.pyc
.pytest_cache
.coverage
htmlcov
.venv
.next
dist
*.md
LICENSE
.vscode
.idea
.DS_Store
docker-compose*.yml
```
**Se não existe .dockerignore, o projeto não está pronto. Ponto.**

#### Dockerfiles production-ready (OBRIGATÓRIO)
Todo Dockerfile DEVE:
- Usar **multi-stage build** — stage de build separado do stage final
- Stage final não ter gcc, dev headers, ou ferramentas de compilacao
- Rodar como **usuario não-root** (`USER appuser`, nunca root)
- Ter `.dockerignore` correspondente
- Copiar deps ANTES do código (cache de layers)
- Nunca ter `--reload`, `npm run dev`, ou qualquer flag de desenvolvimento
- Ter EXPOSE apenas das ports necessarias

Exemplo backend Python:
```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /build
COPY pyproject.toml .
RUN pip install --no-cache-dir --target=/deps .

FROM python:3.12-slim
RUN groupadd -r app && useradd -r -g app app
WORKDIR /app
COPY --from=builder /deps /usr/local/lib/python3.12/site-packages
COPY . .
USER app
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Exemplo frontend Next.js:
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
USER app
EXPOSE 3000
ENV PORT=3000 HOSTNAME="0.0.0.0"
CMD ["node", "server.js"]
```
**Requer** `output: 'standalone'` no `next.config.ts`. Sem isso, `.next/standalone` não existe e o build falha.

#### Entrypoints separados dev/prod
- `entrypoint.sh` (prod): sem `--reload`, sem debug flags
- `entrypoint.dev.sh` (dev): com `--reload`, com debug
- `docker-compose.yml` aponta pro prod
- `docker-compose.override.yml` sobrescreve pro dev (volume mounts, entrypoint dev, ports de debug)

#### Secrets
- Nenhum secret hardcoded em nenhum arquivo (nem "dev defaults" que parecem reais)
- `.env` no `.gitignore` — verificar que esta lá
- `.env.example` com placeholders claros (`change-me-*`, `your-key-here`)
- Nenhum secret no `docker-compose.yml` — tudo via `${VAR}` ou `env_file`
- Nenhum secret em logs de build (build args sao vissiveis em `docker history`)

#### Headers e CORS
- Security headers configurados no backend desde o init
- CORS configurável via env var (nunca hardcoded, nunca `*` como default)

#### Dependências
- Rodar `npm audit` / `pip audit` no init
- Zero vulnerabilidades conhecidas de severidade HIGH ou CRITICAL
- Lock files commitados (package-lock.json, requirements.txt com hashes)

### 7. Monta a fundacao de código

- Auth (se o projeto precisa)
- Schema do banco de dados com migrations
- Estrutura de API (rotas, middleware, tratamento de erros)
- Config de deploy (se o target e conhecido)
- Health check endpoint que verifica dependências (DB, Redis, etc — não só retornar 200)

### 8. Estabelece restrições de custo

- Se usa APIs externas (LLM, etc): definir limites de contexto, evitar calls desnecessarias
- Se tem background jobs: definir frequência justificada
- Documentar custo estimado por operação principal
- Custo x performance e restrição de design, não otimização futura

### 9. Documenta as decisões

Crie ARCHITECTURE.md explicando:
- Stack escolhida e por que (cada ferramenta)
- Decisões arquiteturais e trade-offs
- Como rodar o projeto
- Ports e servicos
- Estrutura de pastas
- Decisões de segurança e por que

### 10. Memory + context-aware operation (HM-default)

Todo projeto HM nasce com:

#### `CLAUDE.md` por projeto
- Raiz do repo. Contexto especifico do produto: stack, decisões cravadas, módulos, glossario do DOMÍNIO.
- **Não confundir** com o `CLAUDE.md` global (em `~/.claude/CLAUDE.md` ou equivalente), que define como o time opera de forma universal.
- Endurecer baseline-ready se aplicavel (ex: cobertura mínima, audit-rls OBRIGATÓRIO em Supabase, perf budget). Nunca afrouxar.

#### `MEMORY.md` + topic files
- Raiz do repo. Resumo vivo do estado atual do projeto. **Limite: ~200 linhas.**
- Acima de 200 linhas: mover detalhe pra `docs/topics/<assunto>.md` e manter só o ponteiro no MEMORY.md.
- Auto-checkpoint: ao fechar bloco substancial (feature entregue, refactor grande, debug demorado), gravar no MEMORY.md ou topic file: licao aprendida, gotcha, decisão tomada, motivo.
- PR aberta = estado "em revisão". Branch local = "trabalho em progresso". **Stash não sobrevive** mais que algumas horas — não usar pra estado importante.

### 11. Configurações obrigatorias por stack

#### Next.js (qualquer projeto Next 13+)
- `devIndicators: false` no `next.config.ts` — o overlay de devtools polui design e atrapalha QA visual. Sempre desabilitar em projetos com bar de design alta.
- Output `standalone` se for rodar via Docker
- `output: "export"` apenas em projetos puramente estaticos

```ts
// next.config.ts
export default {
  devIndicators: false,
  // ...
}
```

#### Projeto Supabase (qualquer projeto que use Supabase ou Postgres via PostgREST)
- **`scripts/audit-rls.sh`** — escaneia migrations procurando `CREATE TABLE` em `public` sem `ENABLE ROW LEVEL SECURITY` + `CREATE POLICY` na mesma migration, e `CREATE VIEW` sem `WITH (security_invoker=true)`. Manter um projeto-template com esse script e copiar no setup.
- **`scripts/git-hooks/pré-commit`** — Hook que roda `audit-rls.sh` antes de aceitar commit que toca `supabase/migrations/`.
- Inscrever no email semanal do Supabase Security Advisor — tratar como ping de produção.
- Gate de fechamento de Sprint: `supabase db advisors --linked --level error` → esperar `No issues found`.
- Ver `/hm-security` DOMÍNIO 14 pra regime completo (inclui esqueleto do `audit-rls.sh`).

#### Tailwind v4 + Turbopack
- **Inline styles** pra design tokens (cores, spacing especifico, gradients). Tailwind v4/Turbopack não resolve classes arbitrarias (`bg-[#3a7a8c]`, `gap-[14px]`) de forma confiavel.
- Tailwind OK pra layout primitivos: `flex`, `grid`, `min-h-screen`, `items-center`, breakpoints.
- HM Design System gera código com inline styles, NUNCA Tailwind classes pra tokens.

#### Tauri / Electron (app desktop com DB local)
- DB em `userData` = **produção pessoal** desde o primeiro `npm run tauri build` que você instala em `/Applications`.
- Backup automático no `before-quit` (snapshot SQLite/Postgres → repo privado ou storage local).
- Reset de profile/facts/DB exige confirmacao explícita por operação. Ver `/hm-data-integrity` v2.

## Padrões

- Toda dependência precisa justificar sua existência
- A estrutura de pastas precisa tornar a arquitetura visível
- Sem código placeholder. Sem comentarios TODO no dia um. Tudo que existe funciona.
- O projeto precisa rodar com sucesso apos o init
- Dados sao sagrados desde o primeiro docker-compose.yml
- **Segurança é fundação, não feature. Se falta .dockerignore, multi-stage build, ou non-root user, o init não esta completo.**

## Output

Apos o init, o fundador deve conseguir:
1. Entender cada escolha técnica e o porque
2. Rodar o projeto com um comando
3. Comecar a construir features sem friccao de setup
4. Saber quanto vai custar rodar em produção
5. **Ter certeza que o projeto e seguro desde o commit zero**
6. Ter `CLAUDE.md` + `MEMORY.md` na raiz cravados, prontos pra acumular contexto
7. Em Supabase: `audit-rls.sh` + pré-commit hook funcionando
8. Em Next.js: zero overlay devtools no canto da tela

## Regras
- Não pergunte "qual framework você quer?" — recomende o melhor e explique por que
- Não faca scaffold de arquivos vazios. Todo arquivo que existe tem conteudo real.
- Não use pacotes deprecated ou sem manutencao
- Não configure o que ainda não é necessário. Escopo pro que o projeto precisa agora.
- Se a descricao do fundador for vaga, faca UMA pergunta de esclarecimento antes de prosseguir
- **Nunca exponha secrets ou ports desnecessarios**
- **Nunca crie um projeto sem .dockerignore, multi-stage Dockerfile, e non-root user**
- **Nunca use `npm run dev` ou `--reload` em Dockerfile de produção**
- Se o projeto vai ter agente AI, arquitete agent-first desde o inicio — não "adiciona agente depois"
- **Apos o init, rodar `/hm-security` L1 pra validar que a fundacao de segurança está sólida**
- **Projeto Supabase sem `audit-rls.sh` + pré-commit hook NÃO esta inicializado** — sem isso, regra de RLS só vive em disciplina, e disciplina falha
- **Next.js com overlay de devtools visível** = init incompleto. Sempre `devIndicators: false`
- **CLAUDE.md + MEMORY.md ausentes na raiz** = init incompleto. Sem memoria viva, contexto se perde entre sessões
