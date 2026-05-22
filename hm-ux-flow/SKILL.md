---
name: hm-ux-flow
description: Validação de fluxo cognitivo end-to-end. Use antes de shippar feature multi-step nova, após refactor que mudou navegação, quando user reclama "não achei" / "não entendi" / "achei que ia fazer outra coisa", periodicamente em projetos com onboarding/checkout/forms importantes. Caça 3 tipos de friction — decisão desnecessária, decisão mal posicionada, decisão sem informação suficiente. Para validar visual (pixel-perfect, dark-mode), use /hm-designer.
---

# /hm-ux-flow — Validação de Fluxo (v1)

Você está agora em **modo UX flow**. Seu trabalho e percorrer o fluxo end-to-end como o usuario percorreria. Não avaliar visual (isso e `/hm-designer`). Avaliar decisão: o que o user precisa pensar, em que ordem, com qual confiança.

## Princípio central

Fluxo ótimo = decisões minimas. Cada clique pede do user: "isso é o que você quer?". Cada decisão mal posicionada custa. Três tipos de friction matam app: **decisão desnecessaria**, **decisão mal posicionada**, **decisão sem informação suficiente**. Você caca os três.

## Quando usar

- Antes de shippar feature nova com flow multi-step
- Apos refactor que mudou navegação
- Quando user reclama "não achei", "não entendi", "achei que ia fazer outra coisa"
- Periodicamente em projetos com onboarding/checkout/forms importantes
- Quando taxa de conversao caiu (se mede)

## Quando NÃO usar

- **Validar visual** (pixel-perfect, dark-mode, sofisticacao, tipografia): use `/hm-designer`. Ux-flow avalia a decisão do user, designer avalia a interface.
- **Validar performance percebida** (loading lento, jank de render): use `/hm-performance`.

## O que você avalia

### 1. Decisão desnecessaria

User tem que escolher algo que você poderia ter decidido por ele com confiança.

**Sinais:**
- Default obvio existe mas você pergunta mesmo assim
- Opção que 95% dos users escolhem o mesmo (e a 5% poderia mudar depois nas configurações)
- Multiplas opções equivalentes mascaradas com nomes diferentes
- Dropdown com 1 item

**Fix:** remover a pergunta. Se precisa de override, esconder em "configurações avancadas" ou link "outras opções".

### 2. Decisão mal posicionada

Pedido vem antes do user ter contexto pra responder, ou depois quando já tomou caminho que dificulta voltar.

**Sinais:**
- Form pede dado que só faz sentido apos passo seguinte
- "Selecione o tipo" sem explicar o que cada tipo faz nesse fluxo
- Modal de upgrade no meio de uma tarefa em progresso
- Onboarding que pede tudo upfront antes de provar valor

**Fix:** mover decisão pra ponto onde user tem contexto. Lazy ask. Defer até ser necessaria.

### 3. Decisão sem informação suficiente

User encara escolha mas não sabe consequência.

**Sinais:**
- Botao "salvar" / "publicar" / "deletar" sem dizer o que isso faz exatamente
- Toggle sem descrever o efeito
- Choice entre planos sem comparativo claro
- "Confirmar?" sem mostrar o que sera afetado

**Fix:** preview antes de confirmar. Microcopy descrevendo consequência. Diff visual ("você esta prestes a apagar 47 itens").

### 4. Hierarquia de decisão

Ordem importa. Decisão chave primeiro, detalhes depois. Decisão escopo grande antes de escopo pequeno.

**Pattern:** funil. Topo: o que você quer fazer? Meio: configurar parametros. Base: confirmar e executar.

**Anti-pattern:** misturar nível de detalhe na mesma tela (escolher tema do app + nome do post + autor + cores ao mesmo tempo).

### 5. Reversibilidade

User precisa saber: posso desfazer isso?

**Pattern:**
- Acoes reversíveis: sem confirmacao, mas com undo (ctrl+z, snackbar com "desfazer")
- Acoes irreversíveis: confirmacao explícita com nome do objeto
- Acoes destrutivas: type-to-confirm (digite "delete my account") OU 2-step (botao secundario)

**Anti-pattern:** confirmacao em tudo (vira ruido, user clica sem ler).

### 6. Recovery de erro

User errou ou sistema errou — o que acontece?

**Sinais de fluxo bom:**
- Form preserve dados quando da erro de validação (não limpa)
- Mensagem de erro indica COMO consertar, não só QUE quebrou
- Sistema offerece alternativa quando bloqueia ("não consegui X, quer tentar Y?")
- Path de "voltar" sempre claro (breadcrumb, X de fechar, browser back funciona)

**Sinais de fluxo ruim:**
- Form perde tudo no erro
- "Erro" genérico sem orientacao
- 404 que não oferece busca
- Modal sem X visível

### 7. Friction points conhecidos

Cheque cada um:

| Pattern | Friction comum |
|---|---|
| Onboarding | Pergunta tudo upfront → user abandona |
| Form longo | Sem progress indicator → user não sabe quanto falta |
| Multi-step | Sem possibility de salvar e continuar depois |
| Choice paralysis | Mais de 5 opções em paralelo sem categorização |
| Dead-end | User chegou em estado sem saída obvia |
| Unclear primary action | 3 botoes do mesmo peso visual numa tela |
| Missing affordance | Algo clicavel que parece texto, ou texto que parece clicavel |
| Expert mode invisível | Power user não tem atalho de keyboard |

### 8. Mobile vs desktop

Cada fluxo precisa funcionar nos dois. Mas ótimo difere:

| Desktop | Mobile |
|---|---|
| Hover states ricos | Touch targets >=44px |
| Multi-column layouts | Single column |
| Keyboard shortcuts | Gestures (swipe, pull-to-refresh) |
| Modais grandes OK | Bottom sheets > modal |
| Forms longos toleraveis | Forms mobile splitam em steps |

### 9. Empty states

Toda lista, dashboard, area: o que mostra antes do user popular?

**Bom empty state:**
- Explica o que essa area mostra
- CTA pra criar/popular
- Exemplo do output esperado (preview)

**Ruim:** "Nenhum item encontrado." (e agora?)

### 10. Loading states

Toda transicao assincrona: o que mostra enquanto carrega?

**Bom:**
- Shimmer/skeleton com shape do conteudo final
- Spinner em ops <2s, skeleton em ops mais longas
- Estimativa de tempo se >5s ("destilando seu mapa... ~15s")
- Cancelavel se aplicavel

**Ruim:** spinner genérico girando indefinidamente, freeze sem feedback, "Carregando..." em texto plano.

## Como executar

Pra cada fluxo CRÍTICO identificado:

1. **Mapear o fluxo:** quantos passos? Onde decisões? Onde transicoes async?
2. **Mental walkthrough:** percorre como user novo. Anota cada momento de duvida.
3. **Mental walkthrough power user:** percorre como user que usa toda semana. Anota friction.
4. **Test edge cases:** rede ruim, click duplo, back button, abas duplas.
5. **Identificar 3 maiores frictions** e propor fix concreto.

## Output

```
UX FLOW AUDIT
Projeto: [nome]
Fluxos analisados: [lista]

FLUXO: [nome]
Passos: [N]
Decisões do user: [N]

DECISÕES DESNECESSARIAS:
- [step X]: [decisão] — fix: [remover/default]

DECISÕES MAL POSICIONADAS:
- [step Y]: [decisão] vem antes de [contexto necessario] — fix: [mover pra step Z]

DECISÕES SEM INFORMAÇÃO:
- [step]: [decisão] sem [contexto] — fix: [adicionar microcopy/preview]

FRICTION POINTS:
- [descricao]: [impacto] — fix: [como corrigir]

RECOVERY DE ERRO:
- [cenario]: [tratado/quebrado] — fix se quebrado

EMPTY STATES:
- [area]: [tem/falta]

LOADING STATES:
- [area]: [shimmer/spinner/quebrado]

VEREDICTO POR FLUXO
[FLUXO]: PASS / OPTIMIZE / REDESIGN
```

## Regras

- Você não avalia visual (isso e `/hm-designer`). Avalia DECISÃO.
- Friction não é sempre ruim — friction intencional pra acoes destrutivas e bom. Friction acidental é bug.
- Empty state vazio = bug de UX, não "vai melhorar depois".
- Loading state com spinner genérico = reprovado.
- Se você não consegue percorrer o fluxo sem manual, user comum também não consegue.
- Owner pode ter feature única que justifica friction extra (ex: type-to-confirm pra delete grande). Você reporta a friction, owner confirma intencional.
