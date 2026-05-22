---
name: hm-validate-all
description: Orquestrador pré-ship que dispara as 5 skills de validação (security, engineer, qa, designer, deploy) em ordem otimizada, consolida findings priorizados em UM único report e declara baseline-ready (Dev Team) ou BLOQUEADO. Use antes de qualquer ship pra produção, após sprint grande, ou quando quer confiança de "está pronto" em uma checagem só. Product-ready continua sendo prerrogativa do Owner.
---

# /hm-validate-all — Validação Completa Pré-Ship (v1.1)

Você está agora em **modo orquestrador**. Seu trabalho e disparar as 5 skills de validação em ordem otimizada, consolidar findings, e entregar UM único report priorizado que diz: **baseline-ready** (Dev Team pode entregar pro Owner) OU **BLOQUEADO**.

## Princípio central

Validação manual passo-a-passo perde tempo. Orquestracao ganha. Mesma barra de qualidade que rodar cada skill isolada, sem o overhead de checkin manual entre cada uma. **Bloqueante ship e tratado como bloqueante. Technical debt e tratado como debt.**

## Esta skill declara baseline-ready, não product-ready

- **Baseline-ready** = Dev Team validou os 5 checks abaixo + auditorias de segurança/engenharia/QA/design/deploy. Skill declara.
- **Product-ready** = Owner validou UX, fit com tese, qualidade visual end-to-end. **Só Owner declara.**

Os 5 checks obrigatorios do baseline-ready (do `~/.claude/CLAUDE.md` global) já sao cobertos pelo `/hm-qa` v4. Esta skill consolida + adiciona security/engineering/designer/deploy:

1. **typecheck verde** — coberto por `/hm-qa` + `/hm-engineer`
2. **lint verde** — coberto por `/hm-qa` + `/hm-engineer`
3. **smoke test rodado pelo Dev Team** — coberto por `/hm-qa`
4. **zero `console.error` em código novo** — coberto por `/hm-qa`
5. **zero TODO CRÍTICO em código novo** — coberto por `/hm-qa`

Se qualquer um dos 5 falhar, veredicto e BLOQUEADO. Sem exceção. Owner não valida produto enquanto baseline não esta verde.

## Quando usar

- Antes de qualquer ship pra produção (interno ou externo)
- Apos sprint grande (multi-features, refactors estruturais)
- Antes de owner aprovar entrega final
- Quando quer confiança de "esta pronto" em uma checagem só

**Não usar pra:** validação parcial (ex: só segurança → use `/hm-security` direto), debug de bug especifico, code review de PR pequena (use `/review`).

## Ordem de execução (importa)

Skills não independem entre si. Ordem certa evita retrabalho:

1. **`/hm-security`** primeiro. Security gate bloqueia tudo. Se tem CRÍTICO, não continua com as outras (perda de tempo).
2. **`/hm-engineer`** depois. Código fundamenta funcionalidade. Bug estrutural invalida QA passing.
3. **`/hm-qa`** terceiro. Funcionalidade validada apos segurança + código OK.
4. **`/hm-designer`** quarto. Design polish vale apos funcional. Inverso = polir UI de feature quebrada.
5. **`/hm-deploy`** último. Deploy gate só importa apos tudo acima OK.

**Exceção:** se o ship e urgente e o owner pediu "valida tudo", roda as 5 mesmo com criticos no caminho — pra ter mapa completo do estado, não pra shippar. Owner decide.

## Como executar

Pra cada skill, use a Skill tool com argumentos especificos do projeto. Os argumentos devem incluir:
- Paths dos módulos/dirs principais
- Foco da validação (ex: "novo módulo astro + multi-rodada do tarot")
- Tipo de distribuicao (Docker / Electron / Vercel / library / etc)

Exemplo de chamadas:

```
Skill(hm-security, "Auditoria L2 do projeto X. Foco: novo módulo Y, integra LLM, processa dados pessoais. Repo em /path/to/repo")

Skill(hm-engineer, "Validar bloco grande de código Z. Foco senior-level, LLM patterns, performance. Repo em /path/to/repo")

Skill(hm-qa, "QA pass pré-uso real. Edge cases, error states, fluxos: A, B, C. Repo em /path/to/repo")

Skill(hm-designer, "Validar interface das telas novas: D, E, F. Padrão Linear/Stripe/A24. Repo em /path/to/repo")

Skill(hm-deploy, "Validar deploy pré-ship. Distribuicao = X. Repo em /path/to/repo")
```

Cada skill retorna findings categorizados por severidade. Você coleta tudo.

## Consolidacao do output

Apos rodar as 5, produza UM report consolidado:

```
/hm-validate-all REPORT
Projeto: [nome]
Data: [data]
Skills rodadas: hm-security, hm-engineer, hm-qa, hm-designer, hm-deploy

BASELINE-READY GATE (5 checks obrigatorios)
1. typecheck: PASS/FAIL
2. lint: PASS/FAIL
3. smoke test (Dev Team rodou): PASS/FAIL/NOT_RUN
4. console.error em código novo: zero/N
5. TODO CRÍTICO em código novo: zero/N
Baseline-ready: SIM / NÃO

EXECUTIVE SUMMARY
Total findings: X CRÍTICO, Y ALTO, Z MEDIO, W BAIXO
Veredicto: BASELINE-READY (pronto pra Owner avaliar product-ready) / BLOQUEADO

BLOQUEANTE SHIP (CRITICOS + ALTOS de segurança)
1. [SKILL] [ID] Titulo curto — fix em 1 linha
2. ...

CORRIGIR ANTES DE USO REAL (ALTOS funcionais)
1. ...

TECHNICAL DEBT ACEITÁVEL (MEDIOS + BAIXOS)
1. ...

PASS (o que está sólido — celebrar de leve)
- [SKILL]: DOMÍNIO X, DOMÍNIO Y, DOMÍNIO Z

PRÓXIMOS PASSOS
1. [Owner action 1]
2. [Owner action 2]
```

### Regras de classificação

- **BLOQUEANTE SHIP**: qualquer CRÍTICO de segurança, qualquer ALTO de segurança, qualquer ALTO funcional que afete fluxo CRÍTICO
- **CORRIGIR ANTES USO REAL**: ALTO funcional em fluxo não-CRÍTICO, MEDIO de segurança que pode virar exploit
- **TECHNICAL DEBT**: MEDIO funcional, BAIXO de qualquer skill
- **PASS**: dominios sem findings

### Severidade reconciliacao

Se duas skills marcaram a mesma issue com severidades diferentes, usa a MAIOR. Ex: hm-security flag MEDIO e hm-engineer marca o mesmo como ALTO → conta como ALTO.

## Otimizações

- Se `/hm-security` retorna CRÍTICO no Security Gate (DOMÍNIO 1 ou 7), **PARA** e reporta. Não roda as outras 4 — perda de tempo. Owner fixa CRÍTICO primeiro.
- Se projeto e novo (sem features grandes recentes), valida as 5 mesmo. Tendência de bugs em código novo > código estável.
- Se projeto e single-user local-only, pula sub-dominios irrelevantes (multi-tenant, file upload público). Documenta no report ("N/A: local single-user").

## Regras

- Você NÃO e a skill de validação. Você DESPACHA outras skills e CONSOLIDA. Não re-faz auditoria que já foi feita.
- **Você declara apenas baseline-ready.** Product-ready e Owner. Nunca pule essa distinção.
- Os 5 checks baseline-ready sao obrigatorios. Se algum falhar = BLOQUEADO. Não tente compensar com "passou nos outros dominios".
- Se uma skill falha tecnicamente (Skill tool error), reporta no consolidated mas não bloqueia as outras.
- Owner pode pedir override ("ship mesmo com X aberto"). Você reporta o estado, owner decide. Não bloqueie autoritariamente — você informa, owner crava.
- Tempo medio: 15-20 min pra projeto medio. Maior se tiver muitos findings pra processar.
- Mantenha o consolidated report **denso e priorizado**, não concatenado raw das 5 outputs.
