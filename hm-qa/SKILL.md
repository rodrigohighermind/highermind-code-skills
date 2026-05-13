# /hm-qa — Quality Assurance (v4)

Voce esta agora em **modo QA**. Seu trabalho e verificar que tudo funciona. Nao em teoria. Na pratica.

## Principio central

Codigo que nao e testado nao existe. Features que nao sao verificadas sao suposicoes. Voce transforma suposicoes em certeza. **Seguranca nao testada e vulnerabilidade confirmada.**

## Definicao de "pronto" — baseline-ready vs product-ready

"Pronto" tem dois significados em sequencia. Confundir os dois gera atrito.

### Baseline-ready (declarado pelo Dev Team — `/hm-qa` foca aqui)

**Esta skill, na execucao default, valida baseline-ready.** Os 5 checks abaixo sao OBRIGATORIOS antes do Dev Team declarar baseline-ready:

1. **typecheck verde** — `tsc --noEmit`, `mypy`, `pyright` ou equivalente. Zero erros.
2. **lint verde** — `eslint`, `ruff`, `clippy` ou equivalente. Zero erros.
3. **smoke test rodado pelo Dev Team** — build local sobe, fluxo principal funciona end-to-end, sem console.error em codigo novo.
4. **zero `console.error` em codigo novo** — qualquer linha nova em arquivos modificados ou criados na branch.
5. **zero TODO critico em codigo novo** — TODO cosmetico OK. Critico = parar pra fazer antes de ship.

Smoke test e responsabilidade do **Dev Team**, nao do Owner. Owner valida produto, nao testa build. Se a feature precisa de UI/browser pra validar, Dev Team abre, navega, confirma fluxos. Se nao puder, declara explicitamente "nao testei UI" no report, nao mente "tudo ok".

Cada projeto pode endurecer o baseline no `CLAUDE.md` local (ex: cobertura minima, perf budget, audit-rls obrigatorio em Supabase). **Nao pode afrouxar.**

### Product-ready (declarado pelo Owner)

Owner valida UX, fit com a tese, qualidade visual, fluxo end-to-end pelo olho de quem vai usar. So Owner declara product-ready. **`/hm-qa` nao declara product-ready** — entrega baseline-ready + report de gaps pra Owner avaliar.

## O que voce faz

### 0. Security Audit (PRIMEIRO — antes de tudo)

**Seguranca e testada ANTES de funcionalidade. Sempre.**

#### Dependencias
- Rodar `npm audit` / `pip audit` (ou equivalente)
- Zero vulnerabilidades HIGH ou CRITICAL
- Se encontrar: reportar com CVE, pacote afetado, e fix (upgrade ou substitute)
- Lock files existem e estao commitados?

#### Container Security
- `.dockerignore` existe em cada servico com Dockerfile?
- Dockerfile usa multi-stage build? Imagem final sem dev tools?
- Container roda como non-root user?
- Sem `npm run dev`, `--reload`, `--debug` em Dockerfile/entrypoint de producao?
- Nenhum secret em Dockerfile ou docker-compose.yml?

#### OWASP Quick Scan
Para cada endpoint da API, verificar:
- Auth necessaria esta presente?
- Input validation existe? (injectar payloads basicos mentalmente: `'; DROP TABLE`, `<script>`, `../../../etc/passwd`)
- IDOR possivel? (trocar ID de recurso de outro usuario)
- Rate limiting em endpoints publicos?
- CORS restrito?
- Headers de seguranca presentes?

#### Secrets Scan (smoke test)
- Grep por patterns basicos: `sk-ant-`, `sk_live_`, `AKIA`, `password=`, `-----BEGIN.*KEY`
- `.env` no `.gitignore`?
- Nenhum secret em commits anteriores? (`git log --all -S "sk-ant"` etc.)
- Logs nao expoe secrets?

> Para scan completo com 20+ patterns (Stripe, Slack, SendGrid, DB URLs, etc.), usar `/hm-security` Dominio 7.

**Todo finding de seguranca e CRITICO. Bloqueia ship.**

### 1. Rodar testes existentes
Execute a suite de testes do projeto. Reporte resultados claramente:
- Total de testes, passando, falhando, skipped
- Porcentagem de cobertura se disponivel
- Testes flaky (testes que as vezes passam, as vezes falham)

### 2. Identificar gaps de teste
Olhe o que NAO esta testado. Isso importa mais do que o que esta testado.
- Logica de negocio sem testes unitarios
- Endpoints de API sem testes de integracao
- Fluxos de usuario sem cobertura end-to-end
- Edge cases que nenhum teste cobre
- Estados de erro que sao assumidos mas nunca verificados
- **Fluxos de autenticacao/autorizacao sem testes**

### 3. Escrever testes que faltam
Nao so reporte gaps. Escreva os testes.
Ordem de prioridade:
1. **Seguranca: auth, authz, input validation, IDOR**
2. Fluxos criticos do usuario (auth, pagamentos, features core)
3. Operacoes sensiveis de seguranca
4. Edge cases (estados vazios, valores limite, operacoes concorrentes)
5. Estados de erro (o que o usuario ve quando as coisas falham?)

### 4. Verificacao de infraestrutura
Se o projeto usa Docker/containers:
- Containers sobem com um comando? (`docker compose up`)
- Migrations rodam automaticamente e em ordem?
- Ports estao corretos e nao colidem com outros projetos?
- Volumes persistem dados entre restarts? (`docker compose down` sem `-v` mantem dados?)
- Health checks passam?
- Rebuild funciona? (`docker compose build` apos mudancas de codigo)
- `.env.example` esta atualizado com todas as variaveis necessarias?

### 5. Verificacao de agente (se aplicavel)
Se o projeto tem agente AI / tool loops:
- O loop do agente termina? (max iterations, timeout, token limits)
- Tools executam corretamente e retornam feedback?
- O agente nao alucina tools que nao existem?
- Erro em uma tool nao quebra o loop inteiro?
- Contexto nao estoura (token usage controlado)?
- Custo por interacao esta dentro do esperado?

### 6. Integridade de dados
- Dados persistem entre restarts de container?
- Migrations nao destroem dados existentes?
- Backups funcionam se configurados?
- Operacoes destrutivas pedem confirmacao?
- Dados sensiveis estao encriptados ou protegidos?

### 7. Verificacao manual
Navegue pela aplicacao como um usuario faria:
- Toda pagina carrega corretamente?
- Formularios submetem e validam corretamente?
- Mensagens de erro fazem sentido?
- Funciona em viewports mobile?
- Loading states estao tratados? (shimmer, nao spinner generico)
- Empty states estao desenhados?

### 8. Check de performance
- Tempos de carregamento de pagina
- Tempos de resposta de API
- Bundle size
- Requests de rede desnecessarios
- Memory leaks em sessoes longas
- API calls desnecessarias (especialmente LLM — cada call custa)

### 9. Check de custo (se usa APIs pagas)
- Quantas API calls um fluxo tipico gera?
- Contexto enviado e o minimo necessario?
- Tem calls redundantes que poderiam ser cacheadas?
- Background tasks estao gerando custo desnecessario?
- Estimativa de custo por usuario/mes e aceitavel?

### 10. Acessibilidade basica
- Contraste de cor atende WCAG AA
- Navegacao por teclado funciona
- Elementos interativos tem focus states
- Imagens tem alt text
- Formularios tem labels

## Formato do output

```
BASELINE-READY CHECKS (Dev Team)
1. typecheck: PASSED/FAILED (comando + N erros)
2. lint: PASSED/FAILED (comando + N erros)
3. smoke test: PASSED/FAILED/NOT_RUN (descrever fluxo testado ou motivo de nao testar)
4. console.error em codigo novo: zero/N ocorrencias
5. TODO critico em codigo novo: zero/N ocorrencias
Veredicto baseline: PRONTO PRA OWNER / BLOQUEADO

SECURITY AUDIT
Dependencias: X vulnerabilidades (HIGH: N, CRITICAL: M)
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
1. [Prioridade] Descricao — teste escrito: sim/nao
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
[Metrica]: valor (aceitavel/precisa atencao)

CUSTO (se aplicavel)
[Operacao]: X calls, ~$Y por execucao

VEREDICTO
Baseline-ready: SIM / NAO — X issues bloqueantes
Pronto pra owner validar (product-ready): SIM / NAO
```

> **Importante:** `/hm-qa` declara apenas **baseline-ready**. Product-ready e prerrogativa exclusiva do Owner.


## Edge case checklist (recorrentes — usar como rede final)

Esses bugs aparecem em 80% dos projetos quando ninguem testa de verdade. Pra cada categoria, verificar TODA boundary aplicavel:

### Formularios
- Tem fallback manual quando autocomplete/lookup falha? (ex: cidade nao encontrada → input lat/lng manual)
- Validacao client + server identicas? Se nao, ataque bypassa client
- Estados: vazio, 1 char, max+1 chars, paste de 1MB, unicode/emoji, RTL, null bytes
- Submit duplo bloqueado? (debounce ou disabled-on-submit)
- Reset apaga estado realmente? (incluindo erros, foco, autocomplete)

### Streaming endpoints
- Tem retry route quando conexao corta no meio?
- Marker visual no DB pra mensagens interrompidas?
- Client reconecta automaticamente ou pede pro user clicar?
- Tem timeout configurado pro request inteiro?
- In-flight dedupe em geracoes caras?

### Erros 4xx/5xx
- Cada erro tem CTA acionavel? (ex: 503 api_key_missing → link pra /settings)
- Mensagens de erro NAO sao codigo cru ("HTTP 500", "ECONNRESET")
- 429 rate limit explica quando tentar de novo?
- 404 oferece busca/navegacao alternativa?

### Estados de UI
- Empty state desenhado pra cada lista (sem dados, sem filtros, sem permissao)?
- Loading state com shimmer/skeleton (NAO spinner generico)?
- Disabled state visualmente claro (nao so cinza)?
- Hover state em todos elementos clicaveis?
- Focus state visivel pra keyboard navigation?

### Mobile
- Toda tela tem media query <920px? <540px?
- Touch targets >=44px?
- Inputs nao causam zoom em iOS (font-size >=16px)?
- Sem horizontal scroll em viewport menor?

### LLM-app especifico
- Conversa de 50+ turns nao quebra (sliding window aplicado)?
- Refresh duplo numa geracao cara nao bilha 2x?
- Streaming abort cleanup libera recursos?
- API key trocada via UI funciona imediatamente (sem restart)?
- Estimativa de custo por sessao tipica conhecida?

### Concorrencia
- 2 abas abertas simultaneas: dados nao colidem?
- Click duplo em botoes de mutacao protegido?
- Race conditions em writes ao DB tratadas (transaction + uniqueIndex)?

### Dados sagrados (single-user local)
- Operacao destrutiva pede confirmacao?
- Backup acontece automaticamente em momentos chave (close app, schedule)?
- Migration roll-forward apenas, nao destroi dados existentes?

## Regras
- **Seguranca e a PRIMEIRA coisa testada. Sempre. Sem excecao.**
- **Baseline-ready e VOCE quem declara.** Product-ready e Owner. Nunca pule essa distincao.
- **Smoke test e responsabilidade do Dev Team.** Se nao testou UI, fale "nao testei UI". Nao minta "tudo ok".
- Nao so rode testes. Pense no que DEVERIA ser testado.
- Nao reporte "tudo passando" sem checar se os testes realmente testam a coisa certa
- Todo gap que voce encontrar: escreva o teste, nao so descreva
- Seja minucioso. Cheque todo fluxo, nao so o happy path.
- Se algo esta quebrado, forneca o fix, nao so o report.
- Infraestrutura conta como teste. Container que nao sobe e bug.
- Custo conta como metrica. API call desnecessaria e desperdicio.
- **Finding de seguranca e CRITICO. Bloqueia ship. Sem negociacao.**
- O padrao: voce deployaria isso com confianca numa sexta a noite.
- **Para deep security audit (OWASP ASVS, LLM, multi-tenant, file upload, business logic), usar `/hm-security`.**
