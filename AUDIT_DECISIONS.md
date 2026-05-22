# Auditoria de Skills HM — 2026-05-22

Auditoria completa do ecossistema Higher Mind Skills. Decisões cravadas + aplicadas em mesma sessão.

**Status:** APLICADO. Commits locais feitos, push pendente de aprovação Owner.

---

## 1. Inventário (estado pré-auditoria)

### Repo `highermind-code-skills` (11 skills + hm-cli untracked)

| Skill | Linhas | Escopo | Quando usar |
|---|---|---|---|
| hm-init | 278 | Início de projeto: stack, estrutura, infra local, segurança day-1 | Novo projeto do zero |
| hm-engineer | 182 | Validação de código senior-level (OWASP quick + arquitetura + perf + LLM) | Pré-ship de bloco de código |
| hm-security | 751 | Auditoria de segurança L1/L2/L3 (CIS Docker, ASVS, supply chain, AI/LLM) | Deep audit pré-deploy externo |
| hm-data-integrity | 218 | Dados sagrados: backup, migration safety, operações destrutivas | Antes de ship que persiste dados |
| hm-qa | 271 | Baseline-ready: typecheck+lint+smoke+console.error+TODO + security quick + functional | Antes de Dev Team declarar baseline-ready |
| hm-llm-guardrails | 364 | Patterns LLM em produção (sliding window, factory, dedupe, abort) | Pré-ship feature LLM |
| hm-performance | 229 | Profiling com metas concretas (LCP, p95, bundle, custo) | Quando lento ou novo módulo crítico |
| hm-designer | 97 | Validação visual (pixel-perfect, dark-first, sofisticação) | Antes de ship visual |
| hm-ux-flow | 193 | Validação de fluxo cognitivo (decisão desnecessária/mal posicionada) | Após refactor de nav ou feature multi-step |
| hm-deploy | 249 | Validação de deploy por distribution model | Antes de subir pra prod |
| hm-validate-all | 133 | Orquestrador: dispara security+engineer+qa+designer+deploy, consolida | Pré-ship pesado |
| hm-cli **NEW** | 587 | Construção de CLI no padrão HM (Bun+Ink+SQLite, agentic-first, cinematográfico) | Novo CLI ou refactor visual |

### Repo `highermind-business-skills` (2 skills, sem mudança)

| Skill | Linhas | Escopo |
|---|---|---|
| hm-align | 76 | Alinhamento de produto: origem (clareza/ruído), timing, valor |
| hm-sequoia | 282 | Guardrail estratégico: Copilot vs Autopilot, Intelligence vs Judgement |

### Outras skills locais

| Skill | Decisão |
|---|---|
| hm-design-system | **DELETADA** (backup em `~/.claude/skills/.archive/`). Estava solta, fora dos repos HM, sem versionamento. Sistemas de design agora vivem dentro de cada projeto (HM Forge no Builder OS, FamilyOS DS no /v2). |
| pm-strategy, pm-technical, pm-ops, pm-data, pm-growth | Migra pra novo repo `highermind-pm-skills` (sprint separada, fora desta auditoria) |

---

## 2. Análise de overlap por par

| Par | Sobreposição | Decisão | Aplicado |
|---|---|---|---|
| hm-init vs hm-engineer | ~20% (segurança day-1) | Refinar fronteira já existente | Sem ação |
| hm-designer vs hm-ux-flow | ~5% | Refinar fronteira | "Quando NÃO usar" em ambos |
| hm-designer vs hm-design-system | zero | Deletar design-system | Aplicado |
| hm-security vs hm-validate-all | zero (orchestrator) | Manter | Sem ação |
| hm-qa vs hm-validate-all | ~15% (qa tem mini security) | Manter (referência já existe) | Sem ação |
| hm-performance vs hm-engineer | ~10% (engineer tem perf curto) | Refinar fronteira | Reference em engineer |
| hm-data-integrity vs hm-security | zero (attack vs bug) | Manter | Sem ação |
| hm-llm-guardrails vs hm-cli | ~5% | Refinar fronteira | Reference em cli |
| hm-cli vs hm-init | zero (especialização) | Manter | Sem ação |
| hm-engineer vs hm-llm-guardrails | ~10% (engineer tem 10 bullets) | Manter (referência já existe) | Sem ação |

---

## 3. Mudanças aplicadas

### 3.1 Skills

- **+1 nova:** `/hm-cli` (587 linhas, hm-cli/SKILL.md)
- **-1 deletada:** `hm-design-system` (backup em `~/.claude/skills/.archive/hm-design-system-20260522.tar.gz`)
- **0 mergeadas, 0 deletadas do HM core**

### 3.2 Refinamentos de fronteira (4 SKILL.md editados)

| Skill | Edit |
|---|---|
| hm-cli | Adicionada referência a `/hm-llm-guardrails` após "Decisões cravadas" |
| hm-designer | Nova seção "Quando NÃO usar" referenciando `/hm-ux-flow` + sistemas de design por projeto |
| hm-ux-flow | Nova seção "Quando NÃO usar" referenciando `/hm-designer` e `/hm-performance` |
| hm-engineer | Adicionada referência a `/hm-performance` após seção Performance |

### 3.3 Docs

- `README.md` — "Cinco modos cognitivos" → "Doze modos cognitivos". Tabela ganhou `/hm-cli`. Fluxo ganhou `/hm-cli`. Comando de desinstalação atualizado pra incluir as 12 skills. Nova seção dedicada `/hm-cli` antes de Instalação.
- `CHANGELOG.md` — Nova entrada `[3.2.0] — 2026-05-22` documentando hm-cli, refinamentos de fronteira, auditoria, deleção de design-system, migração PM-pack.

---

## 4. Antes vs Depois

| | Antes | Depois |
|---|---|---|
| `highermind-code-skills` | 11 skills (+ hm-cli untracked) | 12 skills (hm-cli cravado) |
| `highermind-business-skills` | 2 skills | 2 skills |
| Skills HM totais | 13 | 14 |
| Skills soltas em ~/.claude/skills/ | hm-design-system + PM-pack | PM-pack (a migrar) |

---

## 5. Próximos passos

1. **Push pra GitHub** — `highermind-code-skills` tem 3 commits prontos. **Pendente de aprovação Owner.**
2. **Criar repo `highermind-pm-skills` no GitHub** — 5 skills PM já preparadas localmente. **Pendente: `gh repo create` + push inicial.**
3. **Sprint separada:** revisar PM-pack pelo padrão HM (estrutura, "Quando usar/NÃO usar", tom CFO/banker), atualizar README/CHANGELOG novos do repo PM.
