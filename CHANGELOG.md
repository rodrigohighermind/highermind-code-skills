# Changelog

Todas as mudanças notáveis neste projeto serão documentadas aqui.

Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/).

---

## [3.3.0] — 2026-05-22

Polish run pós-auditoria. Sistema sobe de 8.0/10 médio pra 9.5+/10 corrigindo 5 defeitos transversais que afetavam todas as skills.

### Mudado

#### Frontmatter `name:`/`description:` em 11 SKILL.md
Skills antigas não tinham frontmatter — só título `# /hm-X`. Claude Code descobre skills pela `description:` no frontmatter; sem ele, descoberta degrada pra título genérico. Adicionado em: hm-init, hm-engineer, hm-security, hm-data-integrity, hm-qa, hm-llm-guardrails, hm-performance, hm-deploy, hm-ux-flow, hm-validate-all, hm-designer. (hm-cli já tinha.)

Cada description é 1-2 linhas específicas sobre quando usar + escopo + cross-references pras skills complementares.

#### Acentuação PT-BR completa
1.500+ substituições aplicadas em 11 SKILL.md do code-skills (também em business e pm-skills). 99,6%+ das palavras agora com acentuação completa (não, você, já, também, decisão, segurança, código, etc). Resíduo são casos ambíguos preservados intencionalmente (`esta` demonstrativo vs verbo).

Por que importava: o `~/.claude/CLAUDE.md` global cravava "Maintain full orthographic correctness for pt-br". Sistema violava sua própria regra.

#### Model IDs alinhados com família Claude atual
- "Sonnet 4" genérico → "Sonnet 4.6" em hm-llm-guardrails e hm-engineer
- Confirmadas referências corretas: claude-opus-4-7, Sonnet 4.6, Haiku 4.5

#### Cross-references novas entre skills
- `/hm-init` agora abre apontando pra `/hm-align` + `/hm-sequoia` (alinhamento antes de bootstrap) e pra `/hm-cli` quando projeto é CLI
- `/hm-security` referencia `/hm-data-integrity` (attack-driven vs bug/crash-driven — primos, não gêmeos)
- `/hm-engineer` referencia `/hm-performance` na seção Performance (quick vs deep)
- `/hm-cli` referencia `/hm-llm-guardrails` na seção LLM
- `/hm-designer` ganhou seção "Quando NÃO usar" referenciando `/hm-ux-flow`
- `/hm-ux-flow` ganhou "Quando NÃO usar" referenciando `/hm-designer` + `/hm-performance`

#### Versionamento padronizado nos títulos
Skills sem versão agora têm: `/hm-cli (v1)`, `/hm-designer (v1)`, `/hm-align (v1)`, `/hm-sequoia (v1)`. Repo continua versão global no CHANGELOG; versão individual rastreia evolução de cada skill.

#### `/hm-align` refresh completo
Skill mais fraca do business pack ganhou:
- 3 exemplos concretos de aplicação (alinhado / desalinhado por ego-hype / timing errado)
- Seção "Quando NÃO usar /hm-align" com 5 anti-padrões
- Output template estruturado (ALINHADO / DESALINHADO / INCERTO) substituindo prosa solta
- Cross-reference pra `/hm-sequoia` em decisões de longo prazo

### Decisões preservadas

- `esta` (demonstrativo "esta seção" vs verbo "está pronto") deixado caso-a-caso pra evitar regressão automática. Resíduo verbos `esta` < 0.5% por arquivo.
- `e` (conjunção vs verbo `é`) idem.
- Pluralização ambígua (`nos` oblíquo vs `nós` reto) deixada como estava.

---

## [3.2.0] — 2026-05-22

Auditoria completa do ecossistema HM Skills + nova skill `/hm-cli`. Decisões cravadas em `AUDIT_DECISIONS.md` na raiz do repo.

### Adicionado

#### Nova skill: `/hm-cli` — Construção de CLI no padrão Higher Mind
- Stack obrigatória: **Bun + TypeScript strict + Ink (React no terminal) + bun:sqlite + Anthropic SDK + `bun build --compile`**. Anti-stack documentado (commander.js, chalk, npm em produto local-first).
- Filosofia agentic-first: CLI como agente operando num terminal, não menu de comandos. Conversa é a interface primária.
- Arquitetura 4 camadas de resposta em ordem de custo: slash command (zero token) → intent local regex (zero token) → Sonnet com contexto (~$0.003) → tool call (zero LLM).
- Hierarquia de custo cravada: atalho local $0 → cache de descrição $0 → Haiku 4.5 ~$0.001 → Sonnet 4.6 sem tool ~$0.003 → Sonnet com tools ~$0.01-0.05.
- Patterns canônicos: blocks visuais (border + padEnd/padStart + bars + sparklines), CFO/banker tone PT-BR completo, learning persistente (`source='manual'` sobrescreve LLM), dados sagrados (idempotência + confirmação destrutiva), distribuição via release.
- Working memory truncate (15 turns máx), prompt cache quando aplicável, abort/cancel em toda chamada async.

### Mudado — refinamentos de fronteira

Auditoria identificou 4 pontos onde cross-reference entre skills clarifica quando usar qual. Edits cirúrgicos, sem mudar comportamento das skills:

- **`/hm-cli`** — adicionada nota apontando pra `/hm-llm-guardrails` pra catálogo completo de patterns LLM em produção.
- **`/hm-designer`** — nova seção "Quando NÃO usar": referencia `/hm-ux-flow` pra fluxo cognitivo, e sistema de design do projeto pra construir interface do zero.
- **`/hm-ux-flow`** — nova seção "Quando NÃO usar": referencia `/hm-designer` pra visual e `/hm-performance` pra loading lento/jank.
- **`/hm-engineer`** — adicionada nota apontando pra `/hm-performance` pra profiling profundo com metas concretas.

### Auditoria

- `AUDIT_DECISIONS.md` documenta inventário tabular das 12 skills code-skills + 2 business-skills, análise de overlap por par, e decisões consolidadas.
- **Zero deleções de skills HM.** Todas as 14 têm propósito distinto, validado par a par.
- **`hm-design-system` removida** do `~/.claude/skills/` (estava solta, fora dos repos HM, sem versionamento). Backup em `~/.claude/skills/.archive/hm-design-system-20260522.tar.gz`. Sistemas de design agora vivem dentro de cada projeto (HM Forge no Builder OS, FamilyOS DS no /v2).
- **PM-pack** (pm-strategy, pm-technical, pm-ops, pm-data, pm-growth) será extraído pra repo próprio `highermind-pm-skills` em sprint separada.

### Estado

- 11 skills → 12 skills no `highermind-code-skills` (+1: `/hm-cli`).
- 2 skills no `highermind-business-skills` (sem mudança).

---

## [3.1.1] — 2026-05-13

Polish da v3.1.0. Generalização das regras introduzidas — qualquer contexto específico de projeto sai, regras técnicas ficam.

### Mudado

- `/hm-security` v2.3 — Domínio 14 (Supabase / PostgREST RLS Regime) reforçado com esqueleto de `audit-rls.sh` direto na skill pra quem não tem projeto-template canônico.
- `/hm-data-integrity` v2 — Nível "Produção pessoal" reescrito com exemplos genéricos (SQLite em `userData`/`AppData`).
- `/hm-llm-guardrails` v2 — Pattern 13 (identidade não-mesclável) com casos genéricos: apelido vs nome completo, homônimos, sufixos familiares.
- `/hm-init` v3 — Referências a "projeto-template" abstratas em vez de citar repositório específico.

### Não mudado

Conteúdo técnico das regras é idêntico ao v3.1.0. Versões individuais das skills permanecem (v2.3, v2, v4, v1.1, v2, v3).

---

## [3.1.0] — 2026-05-13

Iteração de aprendizados das últimas sprints. Incidentes reais consolidados como regra estática + automatizada, não disciplina.

### Mudado

#### `/hm-security` v2.2 → v2.3
- **Novo DOMINIO 14: Supabase / PostgREST — RLS Regime Absoluto.** Toda `CREATE TABLE` em `public` exige `ENABLE ROW LEVEL SECURITY` + `CREATE POLICY` na mesma migration. Toda `CREATE VIEW` que toca tabela RLS exige `WITH (security_invoker=true)`.
- Gate de fechamento de Sprint: `supabase db advisors --linked --level error` → esperar `No issues found`.
- Automação por projeto: `scripts/audit-rls.sh` + pre-commit hook.
- Caso especial `service_role`: nunca expor em client.

#### `/hm-data-integrity` v1 → v2
- Novo nível **Produção pessoal** entre Local single-user e Multi-user privado. Cobre app instalado em `/Applications` ou `Program Files` + DB em `userData`/`AppData`.
- Lista canônica de operações destrutivas que exigem confirmação explícita por operação: `docker compose down -v`, `rm -rf` em path não temporário, `git push --force`, `git reset --hard` em branch publicada, `DROP TABLE`, `TRUNCATE`, `DELETE` sem WHERE específico, kill de processo de produção, reset de profile/facts/DB em prod pessoal.
- Autorização é por operação específica na sessão atual, não retroage.

#### `/hm-qa` v3 → v4
- Cravado: skill declara **baseline-ready** (Dev Team), não product-ready (Owner).
- Os 5 checks obrigatórios do baseline-ready listados explicitamente: typecheck, lint, smoke test (Dev Team rodou), zero `console.error` em código novo, zero TODO crítico em código novo.
- Smoke test é responsabilidade do Dev Team. Se não testou UI, dizer "não testei UI" — não mentir "tudo ok".

#### `/hm-validate-all` v1 → v1.1
- Gate de baseline-ready integrado ao report consolidado. Se algum dos 5 checks falhar = BLOQUEADO.
- Veredicto reescrito: "BASELINE-READY (pronto pra Owner avaliar product-ready) / BLOQUEADO".

#### `/hm-llm-guardrails` v1 → v2
- Novo pattern 12: **Custo x performance como restrição de design**. Modelo certo pra cada job (Sonnet pra chat, Haiku pra extraction, Opus pra raciocínio profundo), working memory enxuto, prompt caching, custo por feature monitorado.
- Novo pattern 13: **Identidade não-mesclável sem confirmação**. Nunca merge automático baseado em similaridade textual. Qualificador no nome (apelido, sufixo, sigla) = distinção intencional.

#### `/hm-init` v2 → v3
- Nova seção 10: Memory + context-aware operation. `CLAUDE.md` por projeto + `MEMORY.md` (≤200 linhas) + topic files em `docs/topics/`.
- Nova seção 11: Configurações obrigatórias por stack.
  - **Next.js**: `devIndicators: false` por default.
  - **Supabase**: `audit-rls.sh` + pre-commit hook no setup.
  - **Tailwind v4 + Turbopack**: inline styles pra design tokens.
  - **Tauri/Electron**: DB em userData = produção pessoal. Backup no `before-quit`. Reset exige confirmação.

---

## [3.0.0] — 2026-05-10

LLM-native evolution + 4 skills novas + orquestrador. Motivado por aprendizados de ship real. Onde as skills v2 deixaram passar — chat history sem limite, singleton stale, race condition em geração cara, falta de retry no streaming, falta de fallback manual em forms — agora pegam.

### Adicionado

#### Nova skill: `/hm-validate-all` — orquestrador
- Dispara as 5 skills de validação em ordem otimizada (security → engineer → qa → designer → deploy)
- Consolida findings em UM report priorizado (bloqueante ship / corrigir antes uso real / technical debt aceitável)
- Reconcilia severidades quando duas skills marcam mesmo issue
- Para no Security Gate se CRITICO encontrado (não desperdiça tempo nas outras 4)
- Tempo médio: 15-20 min pra projeto médio

#### Nova skill: `/hm-llm-guardrails` — patterns LLM-app
- 12 patterns obrigatórios pra app que integra Claude/GPT/Gemini
- Sliding window de chat history (corrige bug de context overflow após 30+ turns)
- Lazy client factory (sem singleton stale quando user troca API key em runtime)
- In-flight dedupe pra geração cara (evita double billing em refresh duplo)
- Rate limit por endpoint LLM (anti-runaway cost)
- Streaming abort cleanup + retry route com marker no DB
- Schema validation no response (Zod/Pydantic, sem JSON.parse cego)
- Token budget explícito (nunca sem max_tokens)
- Cross-channel context safety (LLM-A injetando em LLM-B com user_notes adversarial)
- Cost tracking + estimativa por sessão
- Multi-provider failover (L3, opcional)
- Tool calling guardrails (sandbox, max iterations, schema validation)

#### Nova skill: `/hm-data-integrity` — dados sagrados
- Backup strategy (atômico, criptografado, versionado, testado, off-site)
- Migration safety (idempotente, reversível ou roll-forward only, backup antes)
- Operações destrutivas (confirmação, soft-delete default, audit log)
- Runtime integrity (transactions, FK, constraints, idempotency keys)
- Schema validation runtime (JSON.parse + Zod sempre)
- DR plan (RPO/RTO, drill executado, runbook escrito)
- Compliance LGPD/GDPR/HIPAA (right to erasure/access, breach notification)
- File/blob integrity (checksum, versioning, lifecycle)
- Observabilidade pra detectar problemas cedo

#### Nova skill: `/hm-performance` — performance profiling
- Bundle size (alvos por framework: Next, Vite)
- Render performance (Core Web Vitals: LCP, INP, CLS)
- API latency (p50/p95/p99)
- Database (indexes, slow queries, cursor pagination)
- LLM tokens (cost por turn, prompt caching, latency)
- Network (HTTP/2, compression, CDN, cache headers)
- Memory (leaks, listeners, bound caches)
- Build performance (HMR <2s, cold build <60s)
- Tools por stack (bundle-analyzer, React DevTools Profiler, pg_stat_statements, web-vitals)

#### Nova skill: `/hm-ux-flow` — validação de fluxo
- Não substitui `/hm-designer` (visual). Foca em DECISÃO do user.
- Detecta 3 tipos de friction: decisão desnecessária, mal posicionada, sem informação
- Hierarquia de decisão (funil: o que → como → confirmar)
- Reversibilidade (acao reversível vs irreversível vs destrutiva)
- Recovery de erro (form preserve dados, msgs acionáveis, dead-ends)
- Friction points conhecidos (onboarding, choice paralysis, missing affordance)
- Mobile vs desktop (touch targets, gestures, bottom sheets)
- Empty states + loading states (shimmer obrigatório, sem spinner genérico)

### Mudado

#### `/hm-engineer` v3 — LLM patterns no padrão senior
- Padrão senior inegociável ganhou 3 itens: zero singletons stale, zero JSON.parse + cast sem validação, zero history unbounded em LLM
- Nova seção "LLM-app patterns" entre Performance e Custo: 10 patterns recorrentes que scanners não pegam (sliding window, lazy client, in-flight dedupe, streaming abort, cross-channel safety, etc)
- Cross-reference com `/hm-llm-guardrails` pra deep audit

#### `/hm-security` v2.2 — LLM-app gotchas expandidos
- Domínio 12 (AI/LLM Security) ganhou 4 sub-domínios novos:
  - 12.5 PII em prompts ganhou check de "disclaimer transparente ao user"
  - 12.6 Cross-channel context safety (LLM-A → LLM-B, confused deputy mitigation)
  - 12.7 API key lifecycle e singleton stale
  - 12.8 Streaming endpoint safety (abort, resumability, backpressure, in-flight billing)
  - 12.9 Sliding window obrigatório (CRITICO se chat route sem `.limit(N)`)

#### `/hm-deploy` v3 — multi-modelo distribution
- Nova seção 0: Distribution Model (identifica modelo antes da auditoria)
- Checks específicos por modelo: Container/Docker, Serverless/Edge, Desktop (Electron), Mobile (Expo/RN), Library/SDK, CLI tool
- Antes da v3, era 100% Docker-centric e exigia adaptação mental pra outros modelos
- Pula seções não aplicáveis ao modelo (ex: Electron não tem `.dockerignore` → pula Domínio 1.1)

#### `/hm-qa` v3 — edge case checklist
- Nova seção: Edge case checklist (8 categorias × N checks cada)
- Categorias: Formulários (fallback manual!), Streaming endpoints, Erros 4xx/5xx (CTA acionável!), Estados de UI, Mobile, LLM-app, Concorrência, Dados sagrados
- Bugs recorrentes em 80% dos projetos quando ninguém testa de verdade

### Filosofia

A v2 era pra times que sabem o que estão fazendo. A v3 é pra times que sabem o que estão fazendo COM LLM. Padrão senior sobrevive — só ganhou patterns nativos do mundo onde apps têm agente, conversa, e chamada externa cara em todo lugar.

---

## [2.1.0] — 2026-04-08

Security-first evolution. Motivada por falhas reais de container que passaram pela v2: Dockerfile com `npm run dev`, sem `.dockerignore`, `--reload` no entrypoint de produção.

### Adicionado

#### Nova skill: `/hm-security` — Auditoria de segurança dedicada
- Skill world-class baseada em OWASP ASVS 5.0, CIS Benchmarks, SLSA, e metodologias de Tempest/CrowdStrike/Trail of Bits
- 3 níveis de auditoria: L1 (baseline), L2 (enterprise), L3 (critical systems)
- 8 domínios: Container, Aplicação (OWASP Top 10 + API Top 10), Auth, Dados, Dependências, Infra, Logging, Crypto
- Secrets scan automático com patterns
- Supply chain audit (lock files, SBOM, provenance)
- Business logic testing (race conditions, IDOR, privilege escalation)
- Compliance mapping (LGPD, GDPR, PCI-DSS)
- Severidades padronizadas: CRITICO bloqueia, sem exceção

#### Skills existentes — Security gate integrado
- `/hm-deploy`: Security Gate como seção 0 (bloqueante antes de tudo)
- `/hm-engineer`: OWASP Top 10 + Container Security como primeira auditoria
- `/hm-init`: Segurança obrigatória desde o commit zero (.dockerignore, multi-stage, non-root)
- `/hm-qa`: Security Audit como primeira etapa antes de testes

### Mudado

- Segurança movida de "camada de auditoria" para "gate bloqueante" em todas as skills
- Output de todas as skills agora tem seção de segurança no topo

---

## [2.0.0] — 2026-04-03

Evolução completa das skills baseada em aprendizados reais de múltiplos projetos construídos com o framework.

### Adicionado

#### Nova skill: `/hm-deploy`
- Validação completa de infraestrutura, containers e reprodutibilidade
- Checklist de Docker (subida, rebuild, dados sagrados)
- Validação de environment e secrets
- Checklist de database e migrations
- Health checks e monitoramento
- Teste de reprodutibilidade (clone limpo)
- Segurança de deploy (ports, CORS, HTTPS, secrets)

#### `/hm-init` — Framework de decisão de stack
- Tabela de critérios ponderados pra avaliação de stack (fit, performance, custo, maturidade, ecossistema, DX, hiring)
- Anti-patterns de escolha explícitos
- Seção de arquitetura agent-first como default (quando aplicável)
- Infraestrutura local com Docker Compose desde o dia 1
- Restrições de custo como parte do design (API calls, hosting, bandwidth)
- Documentação obrigatória via ARCHITECTURE.md
- Princípio "dados são sagrados" desde o primeiro docker-compose.yml

#### `/hm-engineer` — Padrão senior e novas camadas de auditoria
- Baseline de engenheiro senior inegociável (zero bare except, zero any types, zero fire-and-forget, zero secrets hardcoded, zero queries sem limit)
- Nova camada: **Custo x Performance** (API calls justificadas, contexto mínimo em LLMs, token usage consciente)
- Nova camada: **Dados sagrados** (nenhuma operação destrutiva sem confirmação, volumes nomeados, migrations não-destrutivas)
- Nova camada: **Infraestrutura** (Docker rebuild vs restart, migrations automáticas, health checks, ports)
- Expansão de Performance: I/O paralelo, memoização
- Expansão de Arquitetura: validação de agent loops (max iterations, token limits, timeout)
- Regra: dados em risco é sempre CRÍTICO

#### `/hm-designer` — Agent-first UI e pixel perfect
- Filosofia agent-first: UI = visibilidade + override, não input principal
- Referência A24 adicionada às referências estéticas
- Seção **Pixel perfect**: zero tolerância a desalinhamentos, quebras, cortes
- Padrões técnicos: full-width layout, shimmer (não spinner), dark-first, inline styles quando framework não coopera, transições 200-300ms
- Novos anti-patterns: formulários onde agente deveria executar, spinners genéricos, layout centralizado em telas grandes
- Regra: desalinhamento arquitetural (formulário vs agente) é finding

#### `/hm-qa` — Infraestrutura, agente e custo como teste
- Nova seção: **Verificação de infraestrutura** (containers, migrations, ports, volumes, rebuild, .env)
- Nova seção: **Verificação de agente** (tool loops, alucinação de tools, token usage, custo por interação)
- Nova seção: **Integridade de dados** (persistência, migrations não-destrutivas, backups, operações destrutivas)
- Nova seção: **Check de custo** (API calls por fluxo, contexto mínimo, calls redundantes, custo por usuário/mês)
- Output expandido com seções de Infraestrutura, Agente, Integridade de Dados e Custo

### Mudado

- Contagem de skills: 4 → 5 (adição de `/hm-deploy`)
- SKILL.md parent atualizado pra refletir 5 skills
- Todas as skills agora consideram agent-first como paradigma (quando aplicável)
- Performance expandida de "não ter N+1" pra incluir custo de APIs externas e token management

---

## [1.0.0] — 2026-03-12

Release inicial.

### Skills
- `/hm-init` — Início de projeto com melhores ferramentas e estrutura
- `/hm-engineer` — Validação de código em todas as camadas
- `/hm-designer` — Validação de interface contra o mais alto padrão
- `/hm-qa` — Quality assurance completo

### Infraestrutura
- Setup script com symlinks automáticos
- CLAUDE.md.template como ponto de partida
- Instalação global e por projeto
