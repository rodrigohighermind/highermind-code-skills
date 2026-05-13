# /hm-performance — Performance Profiling (v1)

Voce esta agora em **modo performance**. Seu trabalho e medir e localizar gargalos. Nao especular. Nao adivinhar. Numeros concretos, com fix especifico pra cada metrica fora do alvo.

## Principio central

Performance nao e uma feature. E uma restricao de design. App lento e bug. Latencia alta no LLM e custo alto disfarcado. Bundle inflado e tempo de carga perdido. Cada milissegundo importa porque o usuario sente, mesmo que nao consiga nomear.

## Quando usar

- Antes de shippar feature nova com componente performance-critical (chat, lista grande, dashboard com muitos widgets)
- Quando user reclama "ta lento"
- Apos refactor que mexeu em data flow
- Periodicamente em projetos com escala (semanal/mensal)
- Quando custo de LLM/API explode

## Dominios

### 1. Frontend — Bundle size

| Metrica | Alvo | Critico se |
|---|---|---|
| First-load JS (Next.js) | <200KB gzipped | >400KB |
| Per-route JS | <100KB gzipped | >200KB |
| Total JS no app | <500KB gzipped | >1MB |
| CSS | <50KB gzipped | >100KB |
| Image assets primeira tela | <200KB total | >500KB |

**Como medir:**
```bash
# Next.js
bun run build  # mostra tamanho por rota
# ou
ANALYZE=true bun run build  # com @next/bundle-analyzer

# Vite
bun run build  # mostra warnings pra chunks >500KB

# Generic
du -sh .next/static/chunks/*.js | sort -h | tail
```

**Anti-patterns:**
- Importar lib inteira: `import _ from 'lodash'` (use `lodash-es` named imports)
- Moment.js (use date-fns ou Intl nativo)
- Lib de charts pesada quando precisa de 1 chart simples (recharts vs chart.js vs SVG manual)
- React component lib gigante usada pra 3 componentes (avalia composicao manual)

### 2. Frontend — Render performance

| Metrica | Alvo |
|---|---|
| LCP (Largest Contentful Paint) | <2.5s |
| FID/INP (Interaction to Next Paint) | <200ms |
| CLS (Cumulative Layout Shift) | <0.1 |
| TTFB (Time to First Byte) | <600ms |
| Re-renders desnecessarios | Minimizado via memo/useMemo |

**Como medir:**
- DevTools → Lighthouse (audit local)
- web-vitals package + analytics
- React DevTools Profiler pra re-renders

**Anti-patterns:**
- Inline objects/arrays como props (criam novo ref a cada render → quebra memo)
- useEffect com dependencias erradas (re-roda demais OU stale closure)
- Lista grande sem virtualizacao (>200 items, usa react-window/tanstack-virtual)
- Imagens sem `next/image` (sem optimization, sem lazy)
- Web fonts sem `font-display: swap` (FOIT)

### 3. Backend — API latency

| Metrica | Alvo |
|---|---|
| p50 endpoint normal | <100ms |
| p95 endpoint normal | <300ms |
| p99 endpoint normal | <800ms |
| Endpoint com LLM streaming | First token <2s, total proporcional |
| Endpoint background (analytics, etc) | Aceitavel >1s |

**Como medir:**
- Logging estruturado com `start_time` / `duration_ms` por request
- APM (Sentry, Datadog, BetterStack)
- `time` no curl pra teste manual

**Anti-patterns:**
- N+1 query: 1 query parent + N queries por filho (use JOIN ou batch via `inArray`)
- Query sem index na coluna usada em WHERE/ORDER BY
- Full table scan em tabela >10k rows
- Operacao bloqueante no event loop (Node) — mover pra worker
- Sync I/O em paths quentes

### 4. Database

| Check | Como verificar |
|---|---|
| Indexes nas queries criticas | EXPLAIN ANALYZE no Postgres |
| Conexoes pool dimensionado | pgbouncer ou similar; default ~20 |
| Slow query log ativo | Postgres `log_min_duration_statement = 1000` |
| Vacuum + autoanalyze configurados | Postgres |
| Tables grandes paginadas | LIMIT/OFFSET ou cursor-based |
| JSON columns indexed | GIN/BRIN se queries em JSON path |

**Pattern: cursor-based pagination** pra listas grandes (>1k rows):
```ts
// Nao: OFFSET 50000 (Postgres faz scan ate la)
// Sim: cursor (ultima createdAt vista)
const items = await db
  .select()
  .from(messages)
  .where(lt(messages.createdAt, cursor))
  .orderBy(desc(messages.createdAt))
  .limit(20)
```

### 5. LLM — Token cost + latency

| Metrica | Alvo (Claude Opus 4.7) |
|---|---|
| Tokens input por turn | <3000 (descontado system prompt) |
| Tokens output por turn | 200-2000 (variavel) |
| Cost por turn | <$0.05 chat normal, <$0.30 geracao longa |
| First token latency | <2s |
| Tokens/sec output | 30-80 (Anthropic streaming) |
| Cost por usuario/mes | Estimar e definir alerta |

**Pattern: prompt caching (Anthropic):**
```ts
// System prompt + facts grandes que nao mudam → cache
const response = await anthropic.messages.create({
  model: 'claude-opus-4-7',
  max_tokens: 1024,
  system: [
    { type: 'text', text: largeStaticInstructions, cache_control: { type: 'ephemeral' } },
    { type: 'text', text: dynamicContext }, // nao cacheado
  ],
  messages,
})
```
**Hit rate >50%** = ganho de 90% em custo do prompt + latencia menor. Mede via `usage.cache_read_input_tokens`.

### 6. Network

| Check | Criterio |
|---|---|
| HTTP/2 ou HTTP/3 ativo | TLS verifica via `openssl s_client` |
| Compression (gzip/brotli) | Headers `Content-Encoding` |
| CDN configurado pra assets estaticos | Vercel/Cloudflare |
| Cache headers corretos | `Cache-Control: public, max-age=31536000, immutable` em assets versionados |
| Preconnect/preload pra origens criticas | `<link rel="preconnect">` em fonts/api |

### 7. Memory

| Check | Como medir |
|---|---|
| Heap nao cresce indefinidamente | Heap snapshot via DevTools, Chrome Performance |
| Sem leaks em listeners (cleanup em useEffect) | Inspecionar event listeners |
| Cache com bound (Map sem limite vira leak) | Use LRU cache |
| Streams fechadas | Reader released apos uso |

### 8. Build performance

- Hot reload <2s (Turbopack/Vite)
- Cold build aceitavel (<60s pra projeto medio)
- CI build cache configurado
- Type check incremental (tsc --build se monorepo)

## Profiling tools por stack

| Stack | Tool |
|---|---|
| Next.js bundle | `@next/bundle-analyzer` |
| React renders | React DevTools Profiler |
| Node.js | `clinic.js`, `0x` flamegraphs |
| Postgres | pg_stat_statements, pgBadger, EXPLAIN ANALYZE |
| LLM cost | dashboard interno + Anthropic console |
| Web vitals | web-vitals package, Lighthouse, PageSpeed Insights |
| Network | Chrome DevTools Network tab, WebPageTest |
| Memory | Chrome DevTools Memory tab |

## Output

```
PERF AUDIT
Projeto: [nome]
Profile alvo: [Local app / Internal SaaS / Public SaaS]
Pages/endpoints auditados: [lista]

FRONTEND BUNDLE
[Page]: X KB gzipped (alvo Y) — PASS/FAIL
Maiores chunks: [lista top 5]

FRONTEND RENDER
LCP: X (alvo <2.5s)
INP: X (alvo <200ms)
CLS: X (alvo <0.1)

BACKEND
[Endpoint]: p50 X ms, p95 Y ms, p99 Z ms — PASS/FAIL

DATABASE
Slow queries detectadas: [count]
Indexes faltando: [lista]

LLM (se aplicavel)
Cost por turn: $X (alvo Y)
Cache hit rate: X% (alvo >50%)
First token latency: X s

NETWORK
[Check]: PASS/FAIL

MEMORY
Heap stable: PASS/FAIL

VEREDICTO
Performance OK / OPTIMIZE [lista de areas]
```

## Regras

- Mede antes de otimizar. Sem numeros, vira speculation.
- Bottleneck e onde o numero esta fora do alvo, nao onde voce **acha** que esta.
- Otimizacao prematura e bug. Mas falta de measurement tambem e bug.
- LLM cost sem tracking = nao shippa em escala.
- Bundle size 2x do alvo = bloqueio. User no 3G nao espera.
- Database sem indexes em queries quentes = bloqueio.
- p99 latency >1s em endpoint user-facing = bloqueio.
- Memory leak em background process = bloqueio (vai cair em horas/dias).
