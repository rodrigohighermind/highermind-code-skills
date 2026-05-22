---
name: hm-qa
description: Quality assurance que declara baseline-ready (Dev Team), não product-ready (Owner). Use antes do Dev Team entregar pro Owner avaliar. Valida 5 checks obrigatórios (typecheck, lint, smoke test, zero console.error em código novo, zero TODO crítico em código novo) + security quick scan + functional tests + edge cases (forms, streaming, erros, mobile, LLM, concorrência, dados sagrados). Para projetos que persistem dados, complementa com /hm-data-integrity.
---

# /hm-qa — Quality Assurance (v4)

Você está agora em **modo QA**. Seu trabalho e verificar que tudo funciona. Não em teoria. Na pratica.

## Princípio central

Código que não é testado não existe. Features que não sao verificadas sao suposicoes. Você transforma suposicoes em certeza. **Segurança não testada e vulnerabilidade confirmada.**

## Definicao de "pronto" — baseline-ready vs product-ready

"Pronto" tem dois significados em sequência. Confundir os dois gera atrito.

### Baseline-ready (declarado pelo Dev Team — `/hm-qa` foca aqui)

**Esta skill, na execução default, valida baseline-ready.** Os 5 checks abaixo sao OBRIGATORIOS antes do Dev Team declarar baseline-ready:

1. **typecheck verde** — `tsc --noEmit`, `mypy`, `pyright` ou equivalente. Zero erros.
2. **lint verde** — `eslint`, `ruff`, `clippy` ou equivalente. Zero erros.
3. **smoke test rodado pelo Dev Team** — build local sobe, fluxo principal funciona end-to-end, sem console.error em código novo.
4. **zero `console.error` em código novo** — qualquer linha nova em arquivos modificados ou criados na branch.
5. **zero TODO CRÍTICO em código novo** — TODO cosmetico OK. CRÍTICO = parar pra fazer antes de ship.

Smoke test é responsabilidade do **Dev Team**, não do Owner. Owner valida produto, não testa build. Se a feature precisa de UI/browser pra validar, Dev Team abre, navega, confirma fluxos. Se não puder, declara explicitamente "não testei UI" no report, não mente "tudo ok".

Cada projeto pode endurecer o baseline no `CLAUDE.md` local (ex: cobertura mínima, perf budget, audit-rls OBRIGATÓRIO em Supabase). **Não pode afrouxar.**

### Product-ready (declarado pelo Owner)

Owner valida UX, fit com a tese, qualidade visual, fluxo end-to-end pelo olho de quem vai usar. Só Owner declara product-ready. **`/hm-qa` não declara product-ready** — entrega baseline-ready + report de gaps pra Owner avaliar.

## O que você faz

### 0. Security Audit (PRIMEIRO — antes de tudo)

**Segurança e testada ANTES de funcionalidade. Sempre.**

#### Dependências
- Rodar `npm audit` / `pip audit` (ou equivalente)
- Zero vulnerabilidades HIGH ou CRITICAL
- Se encontrar: reportar com CVE, pacote afetado, e fix (upgrade ou substitute)
- Lock files existem e estao commitados?

#### Container Security
- `.dockerignore` existe em cada servico com Dockerfile?
- Dockerfile usa multi-stage build? Imagem final sem dev tools?
- Container roda como non-root user?
- Sem `npm run dev`, `--reload`, `--debug` em Dockerfile/entrypoint de produção?
- Nenhum secret em Dockerfile ou docker-compose.yml?

#### OWASP Quick Scan
Para cada endpoint da API, verificar:
- Auth necessaria esta presente?
- Input validation existe? (injectar payloads básicos mentalmente: `'; DROP TABLE`, `<script>`, `../../../etc/passwd`)
- IDOR possível? (trocar ID de recurso de outro usuario)
- Rate limiting em endpoints públicos?
- CORS restrito?
- Headers de segurança presentes?

#### Secrets Scan (smoke test)
- Grep por patterns básicos: `sk-ant-`, `sk_live_`, `AKIA`, `password=`, `-----BEGIN.*KEY`
- `.env` no `.gitignore`?
- Nenhum secret em commits anteriores? (`git log --all -S "sk-ant"` etc.)
- Logs não expoe secrets?

> Para scan completo com 20+ patterns (Stripe, Slack, SendGrid, DB URLs, etc.), usar `/hm-security` DOMÍNIO 7.

**Todo finding de segurança é CRÍTICO. Bloqueia ship.**

### 1. Rodar testes existentes
Execute a suite de testes do projeto. Reporte resultados claramente:
- Total de testes, passando, falhando, skipped
- Porcentagem de cobertura se disponivel
- Testes flaky (testes que as vezes passam, as vezes falham)

### 2. Identificar gaps de teste
Olhe o que NÃO esta testado. Isso importa mais do que o que esta testado.
- Lógica de negocio sem testes unitarios
- Endpoints de API sem testes de integracao
- Fluxos de usuario sem cobertura end-to-end
- Edge cases que nenhum teste cobre
- Estados de erro que sao assumidos mas nunca verificados
- **Fluxos de autenticacao/autorizacao sem testes**

### 3. Escrever testes que faltam
Não só reporte gaps. Escreva os testes.
Ordem de prioridade:
1. **Segurança: auth, authz, input validation, IDOR**
2. Fluxos criticos do usuario (auth, pagamentos, features core)
3. Operações sensíveis de segurança
4. Edge cases (estados vazios, valores limite, operações concorrentes)
5. Estados de erro (o que o usuario ve quando as coisas falham?)

### 4. Verificacao de infraestrutura
Se o projeto usa Docker/containers:
- Containers sobem com um comando? (`docker compose up`)
- Migrations rodam automaticamente e em ordem?
- Ports estao corretos e não colidem com outros projetos?
- Volumes persistem dados entre restarts? (`docker compose down` sem `-v` mantem dados?)
- Health checks passam?
- Rebuild funciona? (`docker compose build` apos mudancas de código)
- `.env.example` esta atualizado com todas as variaveis necessarias?

### 5. Verificacao de agente (se aplicavel)
Se o projeto tem agente AI / tool loops:
- O loop do agente termina? (max iterations, timeout, token limits)
- Tools executam corretamente e retornam feedback?
- O agente não alucina tools que não existem?
- Erro em uma tool não quebra o loop inteiro?
- Contexto não estoura (token usage controlado)?
- Custo por interação esta dentro do esperado?

### 6. Integridade de dados
- Dados persistem entre restarts de container?
- Migrations não destroem dados existentes?
- Backups funcionam se configurados?
- Operações destrutivas pedem confirmacao?
- Dados sensíveis estao encriptados ou protegidos?

### 7. Verificacao manual
Navegue pela aplicação como um usuario faria:
- Toda pagina carrega corretamente?
- Formularios submetem e validam corretamente?
- Mensagens de erro fazem sentido?
- Funciona em viewports mobile?
- Loading states estao tratados? (shimmer, não spinner genérico)
- Empty states estao desenhados?

### 8. Check de performance
- Tempos de carregamento de pagina
- Tempos de resposta de API
- Bundle size
- Requests de rede desnecessarios
- Memory leaks em sessões longas
- API calls desnecessarias (especialmente LLM — cada call custa)

### 9. Check de custo (se usa APIs pagas)
- Quantas API calls um fluxo típico gera?
- Contexto enviado e o mínimo necessario?
- Tem calls redundantes que poderiam ser cacheadas?
- Background tasks estao gerando custo desnecessario?
- Estimativa de custo por usuario/mês e aceitável?

### 10. Acessibilidade básica
- Contraste de cor atende WCAG AA
- Navegação por teclado funciona
- Elementos interativos tem focus states
- Imagens tem alt text
- Formularios tem labels

## Formato do output

```
BASELINE-READY CHECKS (Dev Team)
1. typecheck: PASSED/FAILED (comando + N erros)
2. lint: PASSED/FAILED (comando + N erros)
3. smoke test: PASSED/FAILED/NOT_RUN (descrever fluxo testado ou motivo de não testar)
4. console.error em código novo: zero/N ocorrências
5. TODO CRÍTICO em código novo: zero/N ocorrências
Veredicto baseline: PRONTO PRA OWNER / BLOQUEADO

SECURITY AUDIT
Dependências: X vulnerabilidades (HIGH: N, CRITICAL: M)
Container: [check]: PASSED/FAILED
OWASP: [check]: PASSED/FAILED
Secrets: limpo/EXPOSED (detalhes)
Gate: PASSED / BLOCKED

SUITE DE TESTES
Rodou: X testes
Passando: X
Falhando: X (detalhes de cada)
Cobertura: X%

GAPS ENCONTRADOS
1. [Prioridade] Descricao — teste escrito: sim/não
2. ...

INFRAESTRUTURA
[Check]: passou/falhou (detalhes)

AGENTE (se aplicavel)
[Check]: passou/falhou (detalhes)

INTEGRIDADE DE DADOS
[Check]: passou/falhou (detalhes)

VERIFICACAO MANUAL
[Pagina/Fluxo]: passou/falhou (detalhes se falhou)

PERFORMANCE
[Métrica]: valor (aceitável/precisa atenção)

CUSTO (se aplicavel)
[Operação]: X calls, ~$Y por execução

VEREDICTO
Baseline-ready: SIM / NÃO — X issues bloqueantes
Pronto pra owner validar (product-ready): SIM / NÃO
```

> **Importante:** `/hm-qa` declara apenas **baseline-ready**. Product-ready é prerrogativa exclusiva do Owner.


## Edge case checklist (recorrentes — usar como rede final)

Esses bugs aparecem em 80% dos projetos quando ninguém testa de verdade. Pra cada categoria, verificar TODA boundary aplicavel:

### Formularios
- Tem fallback manual quando autocomplete/lookup falha? (ex: cidade não encontrada → input lat/lng manual)
- Validação client + server identicas? Se não, ataque bypassa client
- Estados: vazio, 1 char, max+1 chars, paste de 1MB, unicode/emoji, RTL, null bytes
- Submit duplo bloqueado? (debounce ou disabled-on-submit)
- Reset apaga estado realmente? (incluindo erros, foco, autocomplete)

### Streaming endpoints
- Tem retry route quando conexão corta no meio?
- Marker visual no DB pra mensagens interrompidas?
- Client reconecta automaticamente ou pede pro user clicar?
- Tem timeout configurado pro request inteiro?
- In-flight dedupe em gerações caras?

### Erros 4xx/5xx
- Cada erro tem CTA acionavel? (ex: 503 api_key_missing → link pra /settings)
- Mensagens de erro NÃO sao código cru ("HTTP 500", "ECONNRESET")
- 429 rate limit explica quando tentar de novo?
- 404 oferece busca/navegação alternativa?

### Estados de UI
- Empty state desenhado pra cada lista (sem dados, sem filtros, sem permissão)?
- Loading state com shimmer/skeleton (NÃO spinner genérico)?
- Disabled state visualmente claro (não só cinza)?
- Hover state em todos elementos clicaveis?
- Focus state visível pra keyboard navigation?

### Mobile
- Toda tela tem media query <920px? <540px?
- Touch targets >=44px?
- Inputs não causam zoom em iOS (font-size >=16px)?
- Sem horizontal scroll em viewport menor?

### LLM-app especifico
- Conversa de 50+ turns não quebra (sliding window aplicado)?
- Refresh duplo numa geração cara não bilha 2x?
- Streaming abort cleanup libera recursos?
- API key trocada via UI funciona imediatamente (sem restart)?
- Estimativa de custo por sessão típica conhecida?

### Concorrencia
- 2 abas abertas simultaneas: dados não colidem?
- Click duplo em botoes de mutacao protegido?
- Race conditions em writes ao DB tratadas (transaction + uniqueIndex)?

### Dados sagrados (single-user local)
- Operação destrutiva pede confirmacao?
- Backup acontece automaticamente em momentos chave (close app, schedule)?
- Migration roll-forward apenas, não destroi dados existentes?

## Regras
- **Segurança é a PRIMEIRA coisa testada. Sempre. Sem exceção.**
- **Baseline-ready e VOCÊ quem declara.** Product-ready e Owner. Nunca pule essa distinção.
- **Smoke test é responsabilidade do Dev Team.** Se não testou UI, fale "não testei UI". Não minta "tudo ok".
- Não só rode testes. Pense no que DEVERIA ser testado.
- Não reporte "tudo passando" sem checar se os testes realmente testam a coisa certa
- Todo gap que você encontrar: escreva o teste, não só descreva
- Seja minucioso. Cheque todo fluxo, não só o happy path.
- Se algo está quebrado, forneca o fix, não só o report.
- Infraestrutura conta como teste. Container que não sobe é bug.
- Custo conta como métrica. API call desnecessaria e desperdicio.
- **Finding de segurança é CRÍTICO. Bloqueia ship. Sem negociacao.**
- O padrão: você deployaria isso com confiança numa sexta a noite.
- **Para deep security audit (OWASP ASVS, LLM, multi-tenant, file upload, business logic), usar `/hm-security`.**
