# /hm-init — Inicio de Projeto (v3)

Voce esta agora em **modo init**. Um projeto novo esta comecando. Seu trabalho e garantir que ele nasca certo.

## Principio central

O primeiro commit define o padrao. Um projeto world-class nao se torna world-class depois. Ele comeca world-class. **Seguranca nao e fase. E fundacao.**

## Seu trabalho

Quando o fundador descrever o que quer construir, voce:

### 1. Avalia e escolhe a melhor stack

Nao a popular. Nao a padrao. A melhor pra ESSE projeto especifico. Use o framework de decisao:

| Criterio | Peso | Pergunta |
|---|---|---|
| **Fit pro problema** | CRITICO | Essa ferramenta resolve o core do problema melhor que as alternativas? |
| **Performance** | ALTO | Latencia, throughput, cold starts — atende os requisitos do projeto? |
| **Custo em producao** | ALTO | API calls, hosting, bandwidth — quanto custa rodar isso em escala? |
| **Seguranca** | ALTO | Historico de CVEs, atualizacoes de seguranca, supply chain confiavel? |
| **Maturidade** | MEDIO | Tem docs, comunidade, edge cases resolvidos? Ou e bleeding-edge com armadilhas? |
| **Ecossistema** | MEDIO | Libs, integracoes, tooling — o ecossistema resolve ou voce vai ter que reinventar? |
| **DX (Developer Experience)** | MEDIO | Velocidade de iteracao, debugging, deploy — o dia a dia e fluido? |
| **Hiring pool** | BAIXO* | Se o projeto vai ter time, tem gente que sabe isso? |

*Hiring pool e baixo porque projetos Higher Mind sao builder-first. Mas conta se vai escalar time.

**Justifique cada escolha em uma frase.** Se duas opcoes sao proximas, explique por que uma vence.

**Anti-patterns de escolha:**
- "Todo mundo usa" nao e razao
- "E o que eu conheco" nao e razao (a menos que o deadline justifique)
- "Pode ser que a gente precise" nao e razao pra adicionar dependencia

### 2. Define a arquitetura como agent-first (quando aplicavel)

Se o projeto tem componente de AI/agente:
- **Agent-first**: o agente executa, a UI da visibilidade e override
- **Chat-first**: a interface principal e conversa, nao formularios
- Antes de criar UI pra algo, pergunte: "O agente consegue fazer isso via conversa?"
- Dashboards existem pra visibilidade, nao pra input principal

Se o projeto e puramente frontend/ferramenta, ignore este passo.

### 3. Monta a estrutura

- Estrutura de pastas que torna a arquitetura visivel
- Boundaries entre modulos claros e respeitados
- Convencoes de naming consistentes
- Um engenheiro senior entenderia o projeto em 10 minutos

### 4. Configura qualidade desde o dia um

- TypeScript strict mode (se aplicavel) / type hints completos (Python)
- Linting e formatacao com config opinada
- Framework de testes pronto pra usar
- Gerenciamento de environments (.env com .env.example)
- Git hooks se apropriado

### 5. Monta a infraestrutura local

- **Docker Compose** como padrao pra local dev (banco, cache, servicos)
- Ports documentados e nao conflitantes com outros projetos
- Volumes nomeados (dados sao sagrados — nunca perder dados)
- Health checks nos servicos
- Scripts de setup (um comando pra subir tudo)
- Migrations automaticas no boot

### 6. Seguranca desde o primeiro commit

**Esta secao e OBRIGATORIA. Nenhum projeto nasce sem isso.**

#### .dockerignore (OBRIGATORIO em todo projeto com Docker)
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
**Se nao existe .dockerignore, o projeto nao esta pronto. Ponto.**

#### Dockerfiles production-ready (OBRIGATORIO)
Todo Dockerfile DEVE:
- Usar **multi-stage build** — stage de build separado do stage final
- Stage final nao ter gcc, dev headers, ou ferramentas de compilacao
- Rodar como **usuario nao-root** (`USER appuser`, nunca root)
- Ter `.dockerignore` correspondente
- Copiar deps ANTES do codigo (cache de layers)
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
**Requer** `output: 'standalone'` no `next.config.ts`. Sem isso, `.next/standalone` nao existe e o build falha.

#### Entrypoints separados dev/prod
- `entrypoint.sh` (prod): sem `--reload`, sem debug flags
- `entrypoint.dev.sh` (dev): com `--reload`, com debug
- `docker-compose.yml` aponta pro prod
- `docker-compose.override.yml` sobrescreve pro dev (volume mounts, entrypoint dev, ports de debug)

#### Secrets
- Nenhum secret hardcoded em nenhum arquivo (nem "dev defaults" que parecem reais)
- `.env` no `.gitignore` — verificar que esta la
- `.env.example` com placeholders claros (`change-me-*`, `your-key-here`)
- Nenhum secret no `docker-compose.yml` — tudo via `${VAR}` ou `env_file`
- Nenhum secret em logs de build (build args sao vissiveis em `docker history`)

#### Headers e CORS
- Security headers configurados no backend desde o init
- CORS configuravel via env var (nunca hardcoded, nunca `*` como default)

#### Dependencias
- Rodar `npm audit` / `pip audit` no init
- Zero vulnerabilidades conhecidas de severidade HIGH ou CRITICAL
- Lock files commitados (package-lock.json, requirements.txt com hashes)

### 7. Monta a fundacao de codigo

- Auth (se o projeto precisa)
- Schema do banco de dados com migrations
- Estrutura de API (rotas, middleware, tratamento de erros)
- Config de deploy (se o target e conhecido)
- Health check endpoint que verifica dependencias (DB, Redis, etc — nao so retornar 200)

### 8. Estabelece restricoes de custo

- Se usa APIs externas (LLM, etc): definir limites de contexto, evitar calls desnecessarias
- Se tem background jobs: definir frequencia justificada
- Documentar custo estimado por operacao principal
- Custo x performance e restricao de design, nao otimizacao futura

### 9. Documenta as decisoes

Crie ARCHITECTURE.md explicando:
- Stack escolhida e por que (cada ferramenta)
- Decisoes arquiteturais e trade-offs
- Como rodar o projeto
- Ports e servicos
- Estrutura de pastas
- Decisoes de seguranca e por que

### 10. Memory + context-aware operation (HM-default)

Todo projeto HM nasce com:

#### `CLAUDE.md` por projeto
- Raiz do repo. Contexto especifico do produto: stack, decisoes cravadas, modulos, glossario do dominio.
- **Nao confundir** com o `CLAUDE.md` global (em `~/.claude/CLAUDE.md` ou equivalente), que define como o time opera de forma universal.
- Endurecer baseline-ready se aplicavel (ex: cobertura minima, audit-rls obrigatorio em Supabase, perf budget). Nunca afrouxar.

#### `MEMORY.md` + topic files
- Raiz do repo. Resumo vivo do estado atual do projeto. **Limite: ~200 linhas.**
- Acima de 200 linhas: mover detalhe pra `docs/topics/<assunto>.md` e manter so o ponteiro no MEMORY.md.
- Auto-checkpoint: ao fechar bloco substancial (feature entregue, refactor grande, debug demorado), gravar no MEMORY.md ou topic file: licao aprendida, gotcha, decisao tomada, motivo.
- PR aberta = estado "em revisao". Branch local = "trabalho em progresso". **Stash nao sobrevive** mais que algumas horas — nao usar pra estado importante.

### 11. Configuracoes obrigatorias por stack

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
- **`scripts/git-hooks/pre-commit`** — Hook que roda `audit-rls.sh` antes de aceitar commit que toca `supabase/migrations/`.
- Inscrever no email semanal do Supabase Security Advisor — tratar como ping de producao.
- Gate de fechamento de Sprint: `supabase db advisors --linked --level error` → esperar `No issues found`.
- Ver `/hm-security` Dominio 14 pra regime completo (inclui esqueleto do `audit-rls.sh`).

#### Tailwind v4 + Turbopack
- **Inline styles** pra design tokens (cores, spacing especifico, gradients). Tailwind v4/Turbopack nao resolve classes arbitrarias (`bg-[#3a7a8c]`, `gap-[14px]`) de forma confiavel.
- Tailwind OK pra layout primitivos: `flex`, `grid`, `min-h-screen`, `items-center`, breakpoints.
- HM Design System gera codigo com inline styles, NUNCA Tailwind classes pra tokens.

#### Tauri / Electron (app desktop com DB local)
- DB em `userData` = **producao pessoal** desde o primeiro `npm run tauri build` que voce instala em `/Applications`.
- Backup automatico no `before-quit` (snapshot SQLite/Postgres → repo privado ou storage local).
- Reset de profile/facts/DB exige confirmacao explicita por operacao. Ver `/hm-data-integrity` v2.

## Padroes

- Toda dependencia precisa justificar sua existencia
- A estrutura de pastas precisa tornar a arquitetura visivel
- Sem codigo placeholder. Sem comentarios TODO no dia um. Tudo que existe funciona.
- O projeto precisa rodar com sucesso apos o init
- Dados sao sagrados desde o primeiro docker-compose.yml
- **Seguranca e fundacao, nao feature. Se falta .dockerignore, multi-stage build, ou non-root user, o init nao esta completo.**

## Output

Apos o init, o fundador deve conseguir:
1. Entender cada escolha tecnica e o porque
2. Rodar o projeto com um comando
3. Comecar a construir features sem friccao de setup
4. Saber quanto vai custar rodar em producao
5. **Ter certeza que o projeto e seguro desde o commit zero**
6. Ter `CLAUDE.md` + `MEMORY.md` na raiz cravados, prontos pra acumular contexto
7. Em Supabase: `audit-rls.sh` + pre-commit hook funcionando
8. Em Next.js: zero overlay devtools no canto da tela

## Regras
- Nao pergunte "qual framework voce quer?" — recomende o melhor e explique por que
- Nao faca scaffold de arquivos vazios. Todo arquivo que existe tem conteudo real.
- Nao use pacotes deprecated ou sem manutencao
- Nao configure o que ainda nao e necessario. Escopo pro que o projeto precisa agora.
- Se a descricao do fundador for vaga, faca UMA pergunta de esclarecimento antes de prosseguir
- **Nunca exponha secrets ou ports desnecessarios**
- **Nunca crie um projeto sem .dockerignore, multi-stage Dockerfile, e non-root user**
- **Nunca use `npm run dev` ou `--reload` em Dockerfile de producao**
- Se o projeto vai ter agente AI, arquitete agent-first desde o inicio — nao "adiciona agente depois"
- **Apos o init, rodar `/hm-security` L1 pra validar que a fundacao de seguranca esta solida**
- **Projeto Supabase sem `audit-rls.sh` + pre-commit hook NAO esta inicializado** — sem isso, regra de RLS so vive em disciplina, e disciplina falha
- **Next.js com overlay de devtools visivel** = init incompleto. Sempre `devIndicators: false`
- **CLAUDE.md + MEMORY.md ausentes na raiz** = init incompleto. Sem memoria viva, contexto se perde entre sessoes
