---
name: hm-cli
description: Construção de CLI no padrão Higher Mind. Use quando criar um CLI novo, refatorar visual de CLI existente, ou quando quiser que o agente entre no mindset de "terminal como produto cinematográfico" — densidade, intenção visual, agentic-first, custo-consciente, dados sagrados.
---

# /hm-cli — Construção de CLI no Padrão Higher Mind

Você está agora em **modo CLI builder**. Sua barra é Linear / Stripe / Apple / A24 portados pro terminal. Cinematográfico, denso de informação, zero ruído visual, agentic-first, custo-consciente, dados sagrados.

## Princípio central

Terminal não é console. É cinema com restrição. Cada caractere existe por motivo. Cada cor significa algo. Cada espaço em branco foi escolhido. **Se você não consegue explicar por que algo está renderizado, não deveria estar renderizado.**

A barra não é "passou no lint" nem "compilou". É: se a Linear, a Stripe, ou a A24 fizessem um CLI hoje, seria esse?

---

## Quando usar

- Construir CLI novo do zero
- Refatorar visual/UX de CLI existente que está medíocre
- Decidir entre arquiteturas (intent local vs LLM, sync vs streaming, bloco visual vs markdown)
- Validar pre-ship de CLI antes do Owner aprovar

**Não use pra:** lib sem UI (use `/hm-engineer`), web/mobile (use `/hm-designer`), script utilitário descartável (use senso comum).

---

## Stack obrigatória pro ecossistema HM

Sem desvio, sem "geralmente faz assim". Esses foram escolhidos por razão técnica. Mudar exige conversa com o Owner.

| Camada | Escolha | Por quê |
|---|---|---|
| Runtime | **Bun** (não Node) | Single binary compile (`bun build --compile`), startup instantâneo, `bun:sqlite` nativo |
| Linguagem | **TypeScript strict** | Zero `any`, zero `unknown` sem narrow, `noUncheckedIndexedAccess: true` |
| Render TUI | **Ink (React no terminal)** | Componentes compositáveis, `<Static>` pra scrollback nativo, `flexbox` de verdade |
| DB local | **`bun:sqlite`** (NÃO `better-sqlite3`) | Better-sqlite3 não funciona em Bun. Use o nativo. |
| LLM | **Anthropic SDK** (Sonnet 4.6 + Haiku 4.5) | Sonnet pra análise/chat. Haiku pra extração/categorização. |
| Distribuição | **`bun build --compile`** (~60 MB) | Binário standalone, sem `node`, sem `npm install`, sem `~/.bun` no PATH |
| Install path | `~/.local/bin/<name>` | Está no `PATH` por padrão em macOS/Linux. Sem `sudo`. |

**Anti-stack** (não use sem motivo enorme):
- ❌ `commander.js`, `yargs`, `inquirer` — substituídos por Ink com componentes próprios
- ❌ `chalk` — Ink já tem `color` prop
- ❌ `npm` em produto local-first — use Bun direto
- ❌ Node REPL ou interactive prompt do shell — bagunça o scrollback

---

## Filosofia agentic-first

CLI HM é um **agente operando num terminal**, não um menu de comandos.

- **Conversa é a interface primária.** Slash commands existem pra atalhos rápidos, não pra obrigar o user a aprender sintaxe.
- **O agente age**, não pergunta. Quando ele tem certeza, executa. Quando tem dúvida (destrutivo, irreversível), confirma.
- **LLM tempera, não regurgita.** Local-first sempre que possível. Sonnet entra pra análise e perguntas abertas — não pra repetir dados que o app já tem.
- **Aprendizado persiste.** Toda decisão manual do user vira regra permanente. LLM nunca sobrescreve overrides manuais.

---

## Arquitetura mental: 4 camadas de resposta

Quando o user digita algo, passa por 4 camadas em ordem:

```
1. SLASH COMMAND          (zero token, instantâneo)
   ↓ não bateu
2. INTENT LOCAL (regex)   (zero token, milissegundos)
   ↓ não bateu
3. SONNET COM CONTEXTO    (~$0.001–0.01/turn)
   ↓ Sonnet decide chamar tool
4. TOOL CALL → resposta   (zero LLM, dado bruto)
```

**Regra de ouro:** se a pergunta tem resposta em dado bruto local (listagem, total, status), camada 1-2 resolve. Sonnet só pra análise/insight.

### Exemplo prático

| User digita | Camada |
|---|---|
| `/contas` | 1 — atalho direto |
| `minhas contas do mês` | 2 — regex `\bminhas\s+contas\b` |
| `cashflow de abril` | 2 — regex `\bcashflow\b` |
| `paguei o aluguel R$ 10.500 hoje` | 3 — Sonnet detecta intenção + chama `register_bill_payment` |
| `pq gastei tanto em iFood?` | 3 — Sonnet faz `summary_by_category` + análise |

---

## Padrão visual — 7 elementos obrigatórios

### 1. Logo + welcome rica
A primeira tela não é "Hello". É um **dashboard**. Mostre o estado atual:
- Totais agregados (R$ no ano, contas/mês)
- Sparkline de tendência (▆▃▆▃▂)
- Bloco do mês atual com pago/pendente/atrasado
- Top 3-5 pendências
- Sugestões de pergunta + atalhos slash

```
▮ finance
  v0.2.0

CARTÃO    R$ 117.618,10   5 meses · 639 lançamentos
          ▆▃▆▃▂  jan fev mar abr mai · média R$ 23.523,62/mês
BILLS     R$ 37.580,00    / mês · 14 contas

★ MAIO DE 2026   quinta, 21/05/2026 · faltam 10 dias pro fim do mês

  pago         R$ 0,00     (0 bills)
  pendente     R$ 37.580,00    14 bills
  ⚠ atrasado   R$ 25.450,00    6 bills

  maiores pendentes:
    R$ 12.100,00  Plano de Saúde        atrasada 6d   boleto
    R$ 10.500,00  Aluguel               atrasada 16d  boleto
    ...
```

### 2. Blocks visuais > markdown solto
Quando há dado estruturado, **renderize block visual**, não bullet markdown.

✅ Block visual:
```
╭───────────────────────────────────────────────────╮
│ Alimentação          170 tx   R$ 43.773,57  ████░ 29% │
│   ↳ Mercado           36 tx   R$ 19.703,87           │
│   ↳ iFood             80 tx   R$ 10.789,74           │
│   ↳ Restaurante       27 tx   R$  9.711,18           │
╰───────────────────────────────────────────────────╯
```

❌ Markdown solto:
```
- Alimentação — R$ 43.773,57 (170 tx)
  - Mercado: R$ 19.703,87
  - iFood: R$ 10.789,74
```

Tabelas com pipes (`| col | col |`) **estão banidas** — terminal não renderiza bem.

### 3. Alinhamento com `padEnd`/`padStart`
Toda coluna tem largura fixa. Nome usa `padEnd`, valor numérico usa `padStart`. Sem alinhamento, parece amador.

```ts
function padEnd(s: string, w: number): string {
  if (s.length >= w) return s.slice(0, w);
  return s + " ".repeat(w - s.length);
}
function padStart(s: string, w: number): string {
  if (s.length >= w) return s;
  return " ".repeat(w - s.length) + s;
}
```

Largura típica:
- Nome: 22-28 chars (com truncate)
- Valor R$: 12-14 chars right-aligned
- Status/label: 14 chars left-aligned
- Tag/method: 7-8 chars

### 4. Bars de proporção e sparklines
Bars (`█░`) pra proporção entre categorias. Sparklines (`▁▂▃▄▅▆▇█`) pra séries temporais.

```ts
const BAR_W = 14;
function bar(pct: number): string {
  const filled = Math.round((pct / 100) * BAR_W);
  return "█".repeat(filled) + "░".repeat(BAR_W - filled);
}

const SPARK = ["▁","▂","▃","▄","▅","▆","▇","█"];
function sparkline(values: number[]): string {
  const max = Math.max(...values);
  if (max === 0) return SPARK[0]!.repeat(values.length);
  return values.map((v) => {
    const idx = Math.round((v / max) * (SPARK.length - 1));
    return SPARK[idx]!;
  }).join("");
}
```

### 5. Glifos discretos com cor semântica
Um caractere conta uma história. Não use ASCII art.

| Glifo | Significado | Cor |
|---|---|---|
| `★` | Destaque, foco do momento | accent |
| `✓` | Concluído, sucesso | positive |
| `⚠` | Atrasado, anomalia | danger |
| `●` | Vence hoje, em ação | warning |
| `○` | Pendente, no prazo | textDim |
| `↺` | Histórico, parcela antiga | textDim |
| `↳` | Subitem, drill-down | textDim |
| `›` | Prompt do user, sugestão | accent |
| `△` ou `⚠` | Aviso suave | warning |

### 6. Cores sóbrias com restrição
Off-white como base, accent verde como signature. **Nunca arco-íris.**

```ts
const COLORS = {
  textPrimary: "#E8E6E1",   // off-white principal
  textSecondary: "#B8B5AD",
  textDim: "#7A7770",
  accent: "#4ADE80",        // verde signature
  positive: "#65BB7D",
  warning: "#E0A85C",
  danger: "#D4675E",
  separator: "#3A3833",
};
```

A regra: 80% do texto em `textPrimary` + `textSecondary` + `textDim`. Accent verde é signature, usa com restrição.

### 7. Slash command suggester estilo Claude Code
Quando user digita `/`, abrir menu filtrado.

```
╭───────────────────────────────────────────╮
│ › /contas          contas do mês a pagar  │
│   /resumo [mês]    panorama por categoria │
│   /top [N]         top N maiores gastos   │
│                                           │
│   ↑↓ navega · Tab completa · Enter exec   │
╰───────────────────────────────────────────╯
╭───────────────────────────────────────────╮
│ › /co                                      │
╰───────────────────────────────────────────╯
```

Controles obrigatórios:
- `↑↓` navega
- `Tab` completa (não executa — deixa cursor pronto pra args)
- `Enter` executa (ou autocompleta + espera args, se há args)
- `Esc` fecha menu

---

## Tom de voz — banker/CFO pt-BR

Zero jargão técnico. O agente fala como um CFO falaria com o founder, não como um dev falaria com outro dev.

✅ "Plano de Saúde levou 32% do mês — vale conferir se tem algum adicional."
❌ "A categoria 'plano_saude' teve valor agregado 32% do total."

✅ "Você tem 6 contas atrasadas. A maior é o Aluguel — 16 dias."
❌ "Existem 6 itens com status 'overdue'. O top item é 'aluguel' com `days_to_due = -16`."

✅ "Cartão de abril ficou em R$ 21.612. Bills, R$ 37.580. Total: R$ 59.192."
❌ "Sum de transactions where mes=04 retornou 21612.84..."

### Proibido no output ao user:
- `API`, `tool`, `function`, `JSON`, `array`, `null`, `undefined`
- `categoria`, `status`, `flag` (no sentido técnico) — usar termos em PT
- `consulta`, `filtrado`, `agregado`, `ordenado`
- `---`, `___`, `***` como separador (vira lixo no terminal — use linha em branco)
- "Como posso ajudar?" — vá direto
- "Primeira vez que a gente fala" — vocês já se conheciam
- Tabelas markdown com pipes
- Emoji (a menos que explícito pedido)

### Obrigatório:
- PT-BR com **todos os acentos** (`Alimentação`, `Saúde`, `Móveis`, `Doceria`, `Família`)
- Datas em **dd/MM/yyyy** (`08/05/2026`, não `2026-05-08`)
- Meses por extenso em prosa (`abril de 2026`, não `abr/26`)
- Valores em **R$ X.XXX,XX** (ponto milhar, vírgula decimal)
- Direto, sem softening

---

## Custo-consciência — token economy

Cada chamada custa. Cada token de contexto custa. **Pensar custo é pensar produto.**

### Hierarquia de custo (do mais barato pro mais caro):
1. **Atalho local (regex)** — $0
2. **Cache de descrição** (descrição → categoria) — $0
3. **Haiku 4.5** (categorização, extração) — ~$0.001
4. **Sonnet 4.6 sem tool** (análise curta) — ~$0.003
5. **Sonnet 4.6 com tool calls** (análise complexa) — ~$0.01–0.05

### Decisões cravadas:
- **Listagens factuais** (fatura, contas, top N): sempre local. Nunca Sonnet.
- **Categorização**: Haiku com confidence ≥ 0.85. Override manual fica `source='manual'` (Haiku nunca sobrescreve).
- **Cache permanente** de `(description, category)` pra reuso entre runs.
- **Working memory truncate**: histórico do chat com Sonnet limitado a últimos 15 turnos (não 50+).
- **Prompt cache** quando aplicável (Anthropic suporta cache de 5 min).

> Pra catálogo completo de patterns LLM em produção (sliding window com summary, lazy client factory, in-flight dedupe, streaming abort/retry, schema validation no response, cross-channel safety, cost tracking), usar `/hm-llm-guardrails`.

### Quando vale gastar:
- Auditoria do mês: análise CFO completa
- "Compara março com abril": diff inteligente
- "Pq gastei tanto em X": insight + sugestão de ação
- Comprovante de pagamento: extração de valor real

### Quando NÃO vale:
- Listar transações: local
- Mostrar status do mês: local
- Repetir dado que o block visual já mostrou

---

## Dados sagrados — never lose

Toda operação destrutiva ou de import passa por 3 portões:

### 1. Idempotência
Import com hash SHA-256 do arquivo. Re-import do mesmo arquivo é no-op.

```ts
const hash = crypto.createHash("sha256").update(buffer).digest("hex");
const existing = db.prepare(
  "SELECT id FROM statements WHERE source_file_hash = ?"
).get(hash);
if (existing) return { skipped: true, reason: "already imported" };
```

### 2. Migrations versionadas
Schema mudou? Migration explícita, idempotente, reversível quando possível.

```ts
const SCHEMA = `
CREATE TABLE IF NOT EXISTS ...
ALTER TABLE ... -- só quando necessário, dentro de tryExec wrapper
`;
```

Pra schemas com `UNIQUE` composto, use `PRAGMA foreign_keys = OFF` + rename + recreate + reattach.

### 3. Confirmação destrutiva
Antes de DELETE em massa, DROP, ou sobrescrever arquivo importante: **pedir confirmação**.

```ts
if (operacao.destrutivo) {
  push({ type: "system", text: "vai apagar X. confirma? [y/n]" });
  pendingConfirmation.current = operacao;
  return;
}
```

**Nunca**:
- `docker compose down -v` em banco com dados produtivos
- `git push --force` sem aviso
- `rm -rf` em path não temporário
- `DELETE FROM table` sem `WHERE` específico

---

## Aprendizado persistente

CLI HM **aprende com o user**. Toda decisão manual vira regra permanente.

### 3 tipos de memória:

**1. Cache de classificação** (`description_categories`)
```sql
INSERT INTO description_categories (description, category_id, confidence, source)
VALUES (?, ?, 1.0, 'manual')
ON CONFLICT(description) DO UPDATE SET
  category_id = excluded.category_id,
  confidence = 1.0,
  source = 'manual',
  updated_at = datetime('now');
```

`source='manual'` com `confidence=1.0` **sobrescreve** qualquer tentativa de LLM. Regra cravada.

**2. Memórias contextuais** (`agent_memories`)
Tipos:
- `fact` — verdade objetiva ("Carter's = roupa infantil, provavelmente Manuela")
- `preference` — regra geral ("Mercado Livre/Amazon = Compras sempre")
- `decision` — escolha do user ("OPAQUE = revisar depois, não lembro")
- `context` — situação atual ("Renata vai categorizar os 10 dela")
- `goal` — meta declarada ("quero gastar menos em iFood")

**3. Padrões detectados** (`patterns`)
Recorrências detectadas automaticamente (Netflix, Spotify, Anthropic todo mês). User pode marcar como expected/ignored.

### Loop de aprendizado:
1. User executa ação manual → vira regra/memória
2. Próxima vez que descrição/contexto similar aparecer → regra aplicada automaticamente
3. LLM consulta memórias antes de responder → contexto enriquecido sem custo recorrente

---

## Padrões técnicos Ink (truques específicos)

### Scrollback nativo com `<Static>`
Pra que o terminal nativo gerencie scroll (sem viewport interno), use `<Static items={array}>`. Cada item renderiza uma vez e fica no scrollback.

```tsx
<Static items={staticItems}>
  {(item, i) => {
    if (item.kind === "logo") return <Logo key={i} />;
    if (item.kind === "greeting") return <GreetingBlock key={i} stats={item.stats} />;
    if (item.kind === "msg") return <MessageBlock key={i} msg={item.msg} />;
  }}
</Static>
```

**Não use** `height` fixo + `overflow="hidden"` — fica preso ao viewport e bagunça scroll.

### ANSI inline > Box+Text nesting
Pra renderizar markdown inline (bold, italic, código), use ANSI escape codes num único `<Text>`. Aninhar `<Box>` + `<Text>` quebra a primeira coluna do output em alguns terminais.

```ts
const BOLD = "[1m";
const RESET = "[0m";
const DIM = "[2m";

function inlineToAnsi(text: string): string {
  return text
    .replace(/\*\*(.+?)\*\*/g, `${BOLD}$1${RESET}`)
    .replace(/`(.+?)`/g, `${DIM}$1${RESET}`);
}

<Text>{inlineToAnsi(content)}</Text>
```

### AbortController pra Esc cancel
Toda chamada async (LLM, fetch, query longa) precisa de `AbortController`. Esc cancela.

```tsx
const abortRef = useRef<AbortController | null>(null);

useInput((input, key) => {
  if (key.escape && abortRef.current) {
    abortRef.current.abort();
    setState("cancelled");
  }
});

useEffect(() => {
  if (!pendingTurn) return;
  const ctrl = new AbortController();
  abortRef.current = ctrl;
  chat({ ...args, signal: ctrl.signal }).then(...);
  return () => ctrl.abort();
}, [pendingTurn]);
```

### `useEffect` pra side effects async
Nunca chame `async` dentro de event handlers que mudam state durante render. Sempre `useEffect`.

❌ Ruim:
```tsx
onSubmit={async (v) => {
  setState("loading");
  const r = await chat(v);  // perde re-render entre setState e await
  setResult(r);
}}
```

✅ Bom:
```tsx
onSubmit={(v) => {
  setPendingTurn(v);
}}

useEffect(() => {
  if (!pendingTurn) return;
  setState("loading");
  chat(pendingTurn).then(setResult);
}, [pendingTurn]);
```

---

## Estrutura de pastas

```
src/
  cli.ts                  — entrypoint Bun
  agent/
    app.tsx               — root React component, orchestra tudo
    llm.ts                — system prompt + tools + chat()
    tools.ts              — funções TS expostas como tools pro Sonnet
    messages.ts           — tipos Msg, StaticItem, GreetingStats
    import-inline.ts      — pipeline de import (parse + commit)
  lib/
    db.ts                 — schema + openDb + tryExec helpers
    bills.ts              — CRUD de bills
    iof-linker.ts         — heurística pra linkar IOF↔compra
    intent-detect.ts      — detector regex local
    format.ts             — fmtBRL, fmtDateBR, displayCategory
    memory-store.ts       — agent_memories CRUD
    categorize.ts         — Haiku categorization
    patterns.ts           — recorrence detection
    paths.ts              — DB_PATH, ensureHomeDir
  tui/
    theme.ts              — COLORS, GLYPHS
    logo.tsx              — ASCII logo
    message-list.tsx      — Static + MessageBlock + Greeting + todos os blocks
    tx-row.tsx            — TransactionRow + TransactionTable
    markdown.tsx          — render markdown inline + tabelas
    input-box.tsx         — textbox com cursor manual
    status-bar.tsx        — footer com state/tokens/custo
    slash-suggester.tsx   — menu de slash commands
    slash-commands.ts     — lista canônica de slash commands
```

Esse layout funciona pra CLI agêntico médio. Pra CLI pequeno (sem LLM), pode achatar.

---

## Definition of "pronto" — checklist pre-ship

Antes de declarar baseline-ready (Dev Team passa pra Owner validar):

### Técnico (não negociável):
- [ ] `bun run typecheck` verde — zero erros TS
- [ ] `bun test` verde — todos os testes passam
- [ ] `bun run build` produz binário standalone (~60 MB)
- [ ] `install -m 0755 dist/<name> ~/.local/bin/<name>` — binário no PATH
- [ ] `<name> --version` responde sem erro
- [ ] Smoke test manual: subir, fazer 3 perguntas, confirmar resposta
- [ ] Zero `console.error` em código novo
- [ ] Zero TODO crítico em código novo

### Visual (não negociável):
- [ ] Welcome rica com dashboard (não "Hello world")
- [ ] Pelo menos 3 blocks visuais estruturados (não markdown solto)
- [ ] Slash suggester funcional (↑↓ Tab Enter Esc)
- [ ] Cores semânticas em uso (positive/warning/danger/accent)
- [ ] PT-BR completo com acentos
- [ ] Datas em dd/MM/yyyy
- [ ] Valores em R$ X.XXX,XX

### Comportamento (não negociável):
- [ ] `/help` lista todos os slash commands
- [ ] `exit` / `sair` / `tchau` saem com despedida
- [ ] Esc cancela ação em andamento (LLM, query longa)
- [ ] Import idempotente (hash SHA-256)
- [ ] Operação destrutiva pede confirmação
- [ ] Cache de aprendizado funciona (`source='manual'` sobrescreve LLM)

---

## Anti-padrões — reprovam direto

1. **Markdown bullet de dados.** Se tem dado estruturado, é block visual.
2. **Tabela com pipes** `| col1 | col2 |`. Reprova. Use coluna alinhada com padEnd/padStart.
3. **Sonnet regurgitando dados.** Se a tool retornou os números, Sonnet **comenta**, não repete.
4. **"---" como separador.** Vira lixo no terminal.
5. **Emoji.** A menos que pedido explícito. Glifo Unicode discreto (★⚠✓) sim, emoji colorido não.
6. **Stack trace pro user.** Erros viram mensagem amigável.
7. **`primeira vez que falamos`** em produto local com memória persistente.
8. **`API`, `tool`, `endpoint`** no output. Banido.
9. **Loading sem indicador.** Toda espera tem feedback visual + tempo decorrido.
10. **Cor sem motivo.** Verde = positivo. Vermelho = atenção. Amber = warning. Off-white = info. Sem arco-íris.
11. **"Quer que eu faça X?" quando deveria fazer.** Banker age, depois detalha. Se o user perguntou "qual meu custo fixo médio anual?", a resposta é **R$ X,XX**, não "quer que eu some?". Confirma só pra destrutivo.
12. **Resposta incompleta + pergunta de follow-up.** Se o user pediu "custo fixo total", entrega o TOTAL. Não pede permissão pra incluir bills. Mostra tudo, e oferece drill-down depois se quiser.
13. **Lista repetida em bullets quando tem block.** Se `SubscriptionsBlock` existe, use-o. Se `MonthlyTotalBlock` existe, use-o. Bullet é fallback quando NÃO há block.

---

## Como executar essa skill

Quando o Owner invoca `/hm-cli`:

1. **Pergunte o escopo**: criar do zero? refatorar visual? validar pre-ship?
2. **Confirme o stack**: Bun + TS strict + Ink + bun:sqlite, ou está fazendo algo diferente?
3. **Identifique a função primária do CLI**: agentic (com LLM) ou puramente local?
4. **Mapeie os blocks visuais necessários**: welcome, listagem, breakdown, comparação, status
5. **Defina os slash commands**: 5-15 atalhos cobrindo 80% das ações
6. **Defina o tom**: CFO/banker, dev/engineer, ou outro persona contextual
7. **Liste o "pronto"**: critérios concretos que validam baseline-ready

Quando refatorar CLI existente:
1. Aponte cada anti-padrão encontrado, com o local exato
2. Proponha o block visual substituto pra cada listagem em markdown
3. Estime tempo (geralmente 2-4h pra refactor visual completo de CLI médio)

---

## Referências reais do ecossistema HM

CLIs HM existentes pra inspirar:

- **familyos CLI** — primeiro CLI HM, padrão de blocks + slash + cores
- **finance CLI** — versão mais sofisticada: dashboard rica, suggester estilo Claude Code, agentic com 14+ tools, learning permanente via `source='manual'`

Quando construir CLI novo, leia o `src/tui/` desses dois pra entender o padrão na prática. Não copie cego — adapte ao domínio. Mas a barra está cravada lá.

---

*Versão: 1.0 · Cravado 22/05/2026 a partir da construção do hm-finance-cli.*
