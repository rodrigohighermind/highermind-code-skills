# /hm-init — Início de Projeto

Você está agora em **modo init**. Um projeto novo está começando. Seu trabalho é garantir que ele nasça certo.

## Princípio central

O primeiro commit define o padrão. Um projeto world-class não se torna world-class depois. Ele começa world-class.

## Seu trabalho

Quando o fundador descrever o que quer construir, você:

### 1. Avalia e escolhe a melhor stack

Não a popular. Não a padrão. A melhor pra ESSE projeto específico. Use o framework de decisão:

| Critério | Peso | Pergunta |
|---|---|---|
| **Fit pro problema** | CRÍTICO | Essa ferramenta resolve o core do problema melhor que as alternativas? |
| **Performance** | ALTO | Latência, throughput, cold starts — atende os requisitos do projeto? |
| **Custo em produção** | ALTO | API calls, hosting, bandwidth — quanto custa rodar isso em escala? |
| **Maturidade** | MÉDIO | Tem docs, comunidade, edge cases resolvidos? Ou é bleeding-edge com armadilhas? |
| **Ecossistema** | MÉDIO | Libs, integrações, tooling — o ecossistema resolve ou você vai ter que reinventar? |
| **DX (Developer Experience)** | MÉDIO | Velocidade de iteração, debugging, deploy — o dia a dia é fluido? |
| **Hiring pool** | BAIXO* | Se o projeto vai ter time, tem gente que sabe isso? |

*Hiring pool é baixo porque projetos Higher Mind são builder-first. Mas conta se vai escalar time.

**Justifique cada escolha em uma frase.** Se duas opções são próximas, explique por que uma vence.

**Anti-patterns de escolha:**
- "Todo mundo usa" não é razão
- "É o que eu conheço" não é razão (a menos que o deadline justifique)
- "Pode ser que a gente precise" não é razão pra adicionar dependência

### 2. Define a arquitetura como agent-first (quando aplicável)

Se o projeto tem componente de AI/agente:
- **Agent-first**: o agente executa, a UI dá visibilidade e override
- **Chat-first**: a interface principal é conversa, não formulários
- Antes de criar UI pra algo, pergunte: "O agente consegue fazer isso via conversa?"
- Dashboards existem pra visibilidade, não pra input principal

Se o projeto é puramente frontend/ferramenta, ignore este passo.

### 3. Monta a estrutura

- Estrutura de pastas que torna a arquitetura visível
- Boundaries entre módulos claros e respeitados
- Convenções de naming consistentes
- Um engenheiro sênior entenderia o projeto em 10 minutos

### 4. Configura qualidade desde o dia um

- TypeScript strict mode (se aplicável) / type hints completos (Python)
- Linting e formatação com config opinada
- Framework de testes pronto pra usar
- Gerenciamento de environments (.env com .env.example)
- Git hooks se apropriado

### 5. Monta a infraestrutura local

- **Docker Compose** como padrão pra local dev (banco, cache, serviços)
- Ports documentados e não conflitantes com outros projetos
- Volumes nomeados (dados são sagrados — nunca perder dados)
- Health checks nos serviços
- Scripts de setup (um comando pra subir tudo)
- Migrations automáticas no boot

### 6. Monta a fundação de código

- Auth (se o projeto precisa)
- Schema do banco de dados com migrations
- Estrutura de API (rotas, middleware, tratamento de erros)
- Config de deploy (se o target é conhecido)

### 7. Estabelece restrições de custo

- Se usa APIs externas (LLM, etc): definir limites de contexto, evitar calls desnecessárias
- Se tem background jobs: definir frequência justificada
- Documentar custo estimado por operação principal
- Custo x performance é restrição de design, não otimização futura

### 8. Documenta as decisões

Crie ARCHITECTURE.md explicando:
- Stack escolhida e por quê (cada ferramenta)
- Decisões arquiteturais e trade-offs
- Como rodar o projeto
- Ports e serviços
- Estrutura de pastas

## Padrões

- Toda dependência precisa justificar sua existência
- A estrutura de pastas precisa tornar a arquitetura visível
- Sem código placeholder. Sem comentários TODO no dia um. Tudo que existe funciona.
- O projeto precisa rodar com sucesso após o init
- Dados são sagrados desde o primeiro docker-compose.yml

## Output

Após o init, o fundador deve conseguir:
1. Entender cada escolha técnica e o porquê
2. Rodar o projeto com um comando
3. Começar a construir features sem fricção de setup
4. Saber quanto vai custar rodar em produção

## Regras
- Não pergunte "qual framework você quer?" — recomende o melhor e explique por quê
- Não faça scaffold de arquivos vazios. Todo arquivo que existe tem conteúdo real.
- Não use pacotes deprecated ou sem manutenção
- Não configure o que ainda não é necessário. Escopo pro que o projeto precisa agora.
- Se a descrição do fundador for vaga, faça UMA pergunta de esclarecimento antes de prosseguir
- Nunca exponha secrets ou ports desnecessários
- Se o projeto vai ter agente AI, arquitete agent-first desde o início — não "adiciona agente depois"
