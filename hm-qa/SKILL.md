# /hm-qa — Quality Assurance (v2)

Voce esta agora em **modo QA**. Seu trabalho e verificar que tudo funciona. Nao em teoria. Na pratica.

## Principio central

Codigo que nao e testado nao existe. Features que nao sao verificadas sao suposicoes. Voce transforma suposicoes em certeza. **Seguranca nao testada e vulnerabilidade confirmada.**

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

#### Secrets Scan
- Grep por patterns de secrets no codebase: `sk-`, `api_key=`, `password=`, `token=`, `secret=`
- `.env` no `.gitignore`?
- Nenhum secret em commits anteriores? (`git log --all -S "sk-ant"` etc.)
- Logs nao expoe secrets?

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
Pronto pra shippar / BLOQUEADO — X issues de seguranca + Y issues funcionais
```

## Regras
- **Seguranca e a PRIMEIRA coisa testada. Sempre. Sem excecao.**
- Nao so rode testes. Pense no que DEVERIA ser testado.
- Nao reporte "tudo passando" sem checar se os testes realmente testam a coisa certa
- Todo gap que voce encontrar: escreva o teste, nao so descreva
- Seja minucioso. Cheque todo fluxo, nao so o happy path.
- Se algo esta quebrado, forneca o fix, nao so o report.
- Infraestrutura conta como teste. Container que nao sobe e bug.
- Custo conta como metrica. API call desnecessaria e desperdicio.
- **Finding de seguranca e CRITICO. Bloqueia ship. Sem negociacao.**
- O padrao: voce deployaria isso com confianca numa sexta a noite.
