# Posts de lançamento — v3.0.0 (drafts)

Três tons. Use o que a audiência pede.

---

## Twitter/X (denso, 1 thread)

**Post 1 (hook):**
```
Soltei v3.0 do highermind-code-skills.

10 skills opinionated pro Claude Code, codificadas pra times que sabem o que estão fazendo COM LLM.

Filosofia: padrão senior inegociável. Dados sagrados. Linear/Stripe/A24 em design.

→ github.com/rodrigohighermind/highermind-code-skills
```

**Post 2:**
```
Por que isso importa:

Toda vez que você abre o Claude Code, começa do zero. O agente não sabe sua barra. Você se repete:

"world-class"
"checa segurança"
"design nível Apple"
"roda testes"

Você traduz seu padrão uma vez. Aqui ele fica.
```

**Post 3:**
```
Novo na v3:

/hm-validate-all → orquestra 5 skills em report único
/hm-llm-guardrails → 12 patterns pra app com LLM (sliding window, lazy client, in-flight dedupe…)
/hm-data-integrity → backup, DR, compliance, dados sagrados
/hm-performance → bundle, render, latency, LLM tokens
/hm-ux-flow → DECISÃO do user, não visual
```

**Post 4:**
```
Evoluídas com aprendizados de ship real:

/hm-engineer agora tem 3 patterns LLM no padrão senior inegociável (zero singleton stale, zero JSON.parse cego, zero history unbounded).

/hm-security ganhou cross-channel safety + API key lifecycle + streaming safety.

/hm-deploy não é mais Docker-centric — Electron/Expo/Library/CLI inclusos.
```

**Post 5 (CTA):**
```
MIT, markdown puro, zero install. Clone, plugin, usa.

Funciona com qualquer Claude Code. Funciona com qualquer projeto. Funciona se você sabe que mediocridade não é opção.

github.com/rodrigohighermind/highermind-code-skills
```

---

## Blog post (denso, ~600 palavras)

### Title
```
v3.0 do highermind-code-skills: 10 skills pra Claude Code, opinionated do jeito certo
```

### Body

> Antes de existir uma empresa, existiu uma mente que decidiu construir.

Há cinco meses lancei o `highermind-code-skills` — cinco skills pro Claude Code que codificam o padrão Higher Mind: empresas são extensões da arquitetura interna do fundador. Se o código é mediano, o padrão era mediano.

Hoje saiu a v3.0. Dez skills agora. Cinco novas, quatro evoluídas. O motor: aprendizados de shipar projetos reais com Claude Code todo dia, e sentir onde as skills v2 deixaram passar.

**O que mudou de fato.**

Quando você constrói uma app que integra LLM (Claude, GPT, Gemini), você descobre que existe uma classe inteira de bugs que scanners não pegam:

- Chat history sem limit explode context window depois de 30+ turns
- Singleton de API client criado no module-load não atualiza quando user troca a key em runtime
- Refresh duplo numa geração cara dispara dois billings paralelos
- Stream interrompido deixa user olhando pra "..." infinito sem retomar
- LLM-A injetando contexto no system prompt do LLM-B abre vetor de prompt injection via user_notes

Esses bugs aparecem todos juntos quando você está num projeto sério. As skills v2 não tinham linguagem pra eles. A v3 tem:

- **/hm-llm-guardrails** (skill nova) — 12 patterns obrigatórios pra app LLM-native
- **/hm-engineer v3** — padrão senior inegociável ganhou três itens: zero singleton stale, zero JSON.parse + cast sem validação, zero history unbounded
- **/hm-security v2.2** — Domínio 12 (AI/LLM) ganhou 4 sub-domínios: cross-channel safety, API key lifecycle, streaming safety, sliding window obrigatório

Outras coisas vieram dos projetos paralelos:

- **/hm-deploy v3** virou multi-modelo (Container, Serverless, Electron, Expo, Library, CLI). Antes era 100% Docker-centric. Quando você está shipando uma `.app` Electron, "tem `.dockerignore`?" não ajuda.
- **/hm-qa v3** ganhou edge case checklist (8 categorias de bugs que aparecem em 80% dos projetos quando ninguém testa de verdade)
- **/hm-data-integrity** (skill nova) — dados sagrados como ritual: backup atômico/criptografado/testado/off-site, migration safety, DR plan, compliance LGPD/GDPR
- **/hm-performance** (skill nova) — performance profiling com numbers concretos, alvos por métrica, fix por gargalo
- **/hm-ux-flow** (skill nova) — não substitui designer (visual). Foca em DECISÃO do user. Friction points, hierarquia de decisão, recovery de erro
- **/hm-validate-all** (skill nova) — orquestrador. Dispara 5 skills em ordem otimizada (security → engineer → qa → designer → deploy), reconcilia severidades, produz UM report priorizado

**O que continua igual.**

A filosofia. A v3 é pra times que sabem o que estão fazendo. Opinionated do jeito certo: dados são sagrados, padrão senior é inegociável, design segue Linear/Stripe/A24 e não Bootstrap, mediocridade reprova sem aviso.

Se você quer skills neutras que funcionam em qualquer caso, fork e desopinione. Esse repo é o que a barra Higher Mind pede.

**Pra quem é.**

Você constrói porque não consegue não construir. Usa Claude Code como time de dev. Sabe exatamente como é world-class, mas está cansado de traduzir esse padrão em palavras toda sessão.

Isso codifica o seu padrão uma vez. O agente opera no seu nível desde o primeiro comando.

MIT. Markdown puro. Zero install.

→ github.com/rodrigohighermind/highermind-code-skills

---

## Hacker News (curto, sóbrio)

### Title
```
Show HN: highermind-code-skills v3 – 10 opinionated Claude Code skills
```

### Body
```
Released v3 of highermind-code-skills — 10 markdown skills for Claude Code
that codify a senior-level engineering bar (security-first, LLM-native,
data integrity as ritual, design at Linear/Stripe quality).

What's new in v3, motivated by shipping real LLM apps:

- /hm-llm-guardrails: 12 patterns scanners don't catch (sliding window
  for chat history, lazy client factory to avoid singleton stale, in-flight
  dedupe for expensive generations, streaming abort+retry, schema validation
  on JSON.parse, cross-channel context safety).

- /hm-engineer v3: added 3 senior-level invariants (zero stale singleton,
  zero JSON.parse without schema, zero unbounded history in LLM routes).

- /hm-deploy v3: multi-distribution-model (was Docker-centric; now covers
  Electron, Expo, Vercel/edge, library, CLI).

- /hm-validate-all: orchestrator. Runs the 5 main validation skills in
  optimized order, consolidates findings, prioritizes by ship-blocking
  vs technical debt.

- New: /hm-data-integrity, /hm-performance, /hm-ux-flow.

MIT. Markdown only. No install.

https://github.com/rodrigohighermind/highermind-code-skills

The opinionation is the point — fork to debate it. PRs that ask to soften
the bar will be politely declined.
```

---

## LinkedIn (sóbrio, business-friendly)

```
Lancei v3.0 do highermind-code-skills — biblioteca open source de skills
opinionated pro Claude Code.

Nesses cinco meses, rodando em ~10 projetos diferentes, aprendi que a
barra "world-class" precisa de patterns explícitos pra apps que integram
LLM. A v3 traz isso:

10 skills no total (4 novas + 4 evoluídas + 1 orquestrador):

/hm-llm-guardrails — patterns LLM-app obrigatórios
/hm-data-integrity — backup, DR, compliance
/hm-performanceormance — performance profiling
/hm-ux-flow — DECISÃO do user
/hm-validate-all — orquestrador pré-ship
/hm-engineer v3 — LLM patterns no padrão senior
/hm-security v2.2 — cross-channel safety, API key lifecycle
/hm-deploy v3 — multi-modelo (Container, Electron, Expo, Lib, CLI)
/hm-qa v3 — edge case checklist
/hm-designer — visual cinematográfico

MIT. Markdown puro. Sem install.

→ github.com/rodrigohighermind/highermind-code-skills

A filosofia: padrão senior inegociável. Dados sagrados. Mediocridade
reprova sem aviso.

Pra quem está fazendo apps de verdade com IA generativa e está cansado
de traduzir o padrão em palavras toda sessão.
```
