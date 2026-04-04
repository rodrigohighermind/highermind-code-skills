# highermind-code-skills

**Antes de existir uma empresa, existiu uma mente que decidiu construir.**

Cinco modos cognitivos de execução para o [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Construído em cima da filosofia Higher Mind: empresas são extensões da arquitetura interna do fundador. Se o código é mediano, o padrão era mediano. Se o software é world-class, a mente por trás dele exigiu world-class.

Skills de direção estratégica (`/hm-align`, `/hm-sequoia`) estão em [highermind-business-skills](https://github.com/rodrigohighermind/highermind-business-skills).

Isso não é um pack de prompts. É um padrão, codificado uma vez, pra você nunca mais ter que se repetir.

---

### O problema

Toda vez que você abre o Claude Code, você começa do zero. O agente não sabe qual é a sua barra. Não sabe que "bom o suficiente" não é suficiente. Não sabe que você pensa em décadas, não em sprints. Então você se repete:

- "Faz world-class."
- "Checa segurança. Todas as camadas."
- "O design precisa ser nível Apple."
- "Roda testes. Cobre tudo."
Você fala as mesmas coisas, com palavras diferentes, toda vez. A qualidade do output depende de quão bem você traduz o seu padrão naquele momento específico.

highermind-code-skills resolve isso. Você traduz o seu padrão uma vez. Depois ele está sempre lá.

---

### Como funciona

Duas camadas:

**CLAUDE.md** — sua identidade. Sempre ativo. Todo projeto, toda sessão. O agente sabe quem você é, qual é a sua barra, e que mediocridade não é opção. Você nunca mais precisa explicar isso.

**Skills** — modos cognitivos que você ativa quando precisa. Cada um coloca o agente num mindset específico com um trabalho específico.

| Skill | Quando | O que faz |
| --- | --- | --- |
| `/hm-init` | Início de projeto | Melhores ferramentas, melhor estrutura, melhores práticas. World-class desde o primeiro arquivo. |
| `/hm-engineer` | Validar código | Arquitetura, segurança, performance, custo, qualidade. Todas as camadas. Production-grade. |
| `/hm-designer` | Validar interface | Pra onde o software está indo, não pra onde ele esteve. Sofisticação, encantamento, beleza. |
| `/hm-qa` | Testar tudo | Roda, quebra, verifica que funciona. Edge cases, fluxos, agente, infra, uso real. |
| `/hm-deploy` | Validar deploy | Containers, migrations, reprodutibilidade, segurança. Pronto pra sair do local. |

> Skills de direção (`/hm-align`, `/hm-sequoia`) estão em [highermind-business-skills](https://github.com/rodrigohighermind/highermind-business-skills).

---

### O fluxo

```
/hm-init       você começa um projeto novo. ele nasce certo.
[constrói]     você dirige, o agente executa.
/hm-designer   você valida se a interface está na barra.
/hm-engineer   você valida se o código está na barra.
/hm-qa         você verifica que realmente funciona.
/hm-deploy     você valida que sobe, roda e reproduz.
```

Você não precisa usar as quatro toda vez. Use o que o momento pede. `/hm-engineer` e `/hm-designer` podem rodar várias vezes enquanto você itera.

> Pra validar direção e estratégia, use as [business skills](https://github.com/rodrigohighermind/highermind-business-skills): `/hm-align` (é a coisa certa?) e `/hm-sequoia` (está indo pro futuro?).

---

### Demo

```
Você:  Preciso construir o fluxo de aplicação pra uma comunidade de founders.
       Membros aplicam, são avaliados, aceitos ou colocados em waitlist.

Você:  /hm-init

Claude: [Seleciona Next.js 15, Tailwind, Supabase, Resend pra emails.
        Estrutura monorepo. Configura auth, schema do banco, rotas de API,
        framework de testes. Tudo organizado, production-grade
        desde o primeiro commit.]

Você:  Constrói o fluxo de aplicação.

Claude: [Implementa: formulário multi-step, fila de revisão, motor de
        decisão, notificações por email, tracking de status, lógica
        de reaplicação.]

Você:  /hm-designer

Claude: O formulário usa layout padrão com cards e bordas cinza.
        Isso é software de 2020. Problemas:
        - Sem hierarquia visual. Toda pergunta tem o mesmo peso.
        - Tipografia padrão. Sem intenção editorial.
        - Fundo claro, sem dark mode.
        - Parece um onboarding de SaaS, não um gateway.

        Correção: uma pergunta por tela, interface dark, transições
        cinematográficas, tipografia editorial. A aplicação deve parecer
        uma experiência, não um formulário.

Você:  /hm-engineer

Claude: [CRÍTICO] Race condition: revisores concorrentes podem aceitar
        o mesmo aplicante, disparando emails de boas-vindas duplicados.
        Fix: optimistic locking no status de revisão.

        [CRÍTICO] Sem rate limiting no endpoint de aplicação.
        Fix: middleware de rate limiter, 3 submissões por email por dia.

        [ALTO] Fila de revisão expõe emails dos aplicantes pra todos
        os revisores sem controle de acesso por role.
        Fix: mascarar emails, adicionar modelo de permissão por revisor.

        Arquitetura sólida. Data flow limpo.
        3 findings. Corrigir os críticos antes de shippar.

Você:  [corrige tudo]

Você:  /hm-qa

Claude: [Roda suite de testes completa. Navega fluxo de aplicação end to end.
        Testa: submissão, submissão duplicada, campos vazios, aceitar,
        rejeitar, waitlist, reaplicação, revisões concorrentes,
        entrega de email, viewport mobile.]

        12/12 cenários passaram. Fluxo de aplicação funciona end to end.
        Um issue menor: tela de sucesso não renderiza corretamente
        em viewports abaixo de 375px. Não bloqueia.
```

---

### Pra quem é isso

Você constrói porque não consegue não construir. Você usa Claude Code como seu time de dev. Você sabe exatamente como é world-class, mas está cansado de traduzir esse padrão em palavras toda sessão.

Isso codifica o seu padrão uma vez. O agente opera no seu nível desde o primeiro comando.

---

## `/hm-init`

**Início de projeto.**

Você abre um projeto novo. Digita `/hm-init` e descreve o que quer construir. O agente não faz só scaffold. Ele toma decisões:

- O melhor framework pra esse tipo de projeto (com framework de decisão ponderado)
- O melhor banco de dados, ORM, solução de auth
- A melhor estrutura de pastas e patterns de arquitetura
- Agent-first como default arquitetural (quando aplicável)
- Infraestrutura local com Docker Compose desde o dia 1
- Restrições de custo como parte do design
- Setup de testes desde o dia um
- Gerenciamento de environments
- Formatação e linting do código

Cada escolha é justificada contra critérios explícitos: fit pro problema, performance, custo em produção, maturidade, ecossistema, DX. Nada é padrão. Nada é "a gente geralmente usa isso."

O padrão: se um time de engenharia world-class olhasse pra esse projeto no dia um, diria "é assim que se começa um projeto."

---

## `/hm-engineer`

**Validar código.**

Você digita `/hm-engineer` e o agente audita tudo. Não é lint. Não é estilo. É estrutura, segurança, custo e resiliência.

Começa com um baseline inegociável de engenheiro senior (zero bare except, zero any types, zero fire-and-forget, zero secrets hardcoded). Depois audita:

- **Arquitetura**: responsabilidades, boundaries, data flow, agent loops
- **Segurança**: injection, auth bypass, secrets, trust boundaries, CSRF, ports expostos
- **Performance**: N+1 queries, indexes, re-renders, bundle size, caching, I/O paralelo
- **Custo x Performance**: API calls justificadas, contexto mínimo em LLMs, token usage consciente
- **Dados sagrados**: nenhuma operação destrutiva sem confirmação, volumes nomeados, migrations seguras
- **Infraestrutura**: Docker rebuild vs restart, health checks, ports, migrations
- **Resiliência**: tratamento de erros, retry logic, failure modes, race conditions
- **Qualidade**: testes significativos, naming, abstrações, dependências
- **Escala**: gargalos em 10x e 100x

O padrão: se você estivesse vendendo esse software e o comprador contratasse engenheiros pra auditar, eles não encontrariam nada pra reclamar.

---

## `/hm-designer`

**Validar interface.**

Não é "faz bonito." É visão.

O design de software está se movendo. O que parecia moderno em 2020 parece datado agora. O que parece moderno agora vai parecer datado em 2028. `/hm-designer` não valida contra o padrão de hoje. Valida contra pra onde o software está indo.

A barra:

- **Sofisticação**: cada elemento tem uma razão pra existir. Nada decorativo. Nada de encher espaço.
- **Diferenciação**: essa interface só poderia pertencer a esse produto. Não é um template com conteúdo trocado.
- **Experiência**: usar esse software precisa parecer alguma coisa. Não neutro. Não invisível. Intencional.
- **Encantamento**: momentos que fazem a pessoa pausar e pensar "isso é lindamente feito."
- **Usabilidade**: sem esforço. O usuário nunca fica na dúvida do que fazer.
- **Beleza**: baseado nos produtos mais belos que a humanidade construiu. Apple. Airbnb. Linear. Stripe. Vercel.

O que é rejeitado:
- Qualquer coisa que parece que foi construída de um template
- Qualquer coisa que poderia pertencer a qualquer produto
- Qualquer coisa que usa light mode sem considerar dark mode
- Qualquer coisa com tipografia padrão, espaçamento padrão, tudo padrão
- Qualquer coisa que prioriza "shippar rápido" em vez de "shippar certo"

O padrão: mostre essa interface pra alguém com gosto. Não um designer. Alguém com gosto. Essa pessoa deve sentir que quem construiu isso se importa profundamente com o craft.

---

## `/hm-qa`

**Testar tudo.**

Código que não é testado não existe. `/hm-qa` roda tudo:

- Testes unitários pra lógica de negócio
- Testes de integração pra endpoints de API e fluxos de dados
- Testes end-to-end pros fluxos críticos do usuário
- Testes de edge case (estados vazios, valores limites, operações concorrentes)
- **Verificação de infraestrutura** (containers sobem? migrations rodam? ports corretos? dados persistem?)
- **Verificação de agente** (tool loops terminam? não alucina tools? custo por interação?)
- **Integridade de dados** (persistência entre restarts, migrations não-destrutivas, backups)
- **Check de custo** (API calls por fluxo, contexto mínimo, custo por usuário/mês)
- Verificação em viewport mobile
- Testes de performance e acessibilidade básica

O agente não só roda testes. Ele pensa no que deveria ser testado e não está. Encontra os gaps.

O padrão: você deployaria isso com confiança numa sexta à noite.

---

## `/hm-deploy`

**Validar deploy e infraestrutura.**

Você digita `/hm-deploy` e o agente valida que o projeto é reprodutível, seguro e pronto pra sair do local.

O que ele checa:

- **Docker**: containers sobem, ficam healthy, rebuild funciona, dados estão protegidos
- **Environment**: .env.example completo, nenhum secret exposto, ports documentados
- **Database**: migrations automáticas, schema consistente, dados persistentes
- **Health**: endpoints de health check, monitoramento de dependências
- **Reprodutibilidade**: clone limpo funciona com um comando
- **Segurança**: ports mínimos, CORS correto, secrets fora de logs

O teste definitivo: um engenheiro novo entra no time na segunda e tem o projeto rodando antes do almoço.

---

## Instalação

**Requisitos:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/).

### Passo 1: Instalar as skills

Abra o Claude Code e rode:

```
git clone https://github.com/rodrigohighermind/highermind-code-skills.git ~/.claude/skills/highermind-code-skills && cd ~/.claude/skills/highermind-code-skills && chmod +x setup && ./setup
```

### Passo 2: Configurar o CLAUDE.md

Copie o template incluído pra `~/.claude/CLAUDE.md` e customize:

```
cp ~/.claude/skills/highermind-code-skills/CLAUDE.md.template ~/.claude/CLAUDE.md
```

Edite `~/.claude/CLAUDE.md` pra adicionar seu nome, seus projetos e suas especificidades.

### O que é instalado

- Arquivos de skill em `~/.claude/skills/highermind-code-skills/`
- Symlinks em `~/.claude/skills/hm-init`, `~/.claude/skills/hm-engineer`, `~/.claude/skills/hm-deploy`, etc.
- `CLAUDE.md.template` como ponto de partida pro seu arquivo de identidade global

Tudo fica dentro de `~/.claude/`. Nada toca seu PATH ou roda em background.

---

## Adicionar a um projeto (opcional)

Pra compartilhar as skills com o time num repo específico:

```
cp -Rf ~/.claude/skills/highermind-code-skills .claude/skills/highermind-code-skills && rm -rf .claude/skills/highermind-code-skills/.git && cd .claude/skills/highermind-code-skills && ./setup
```

---

## Atualização

```
cd ~/.claude/skills/highermind-code-skills && git fetch origin && git reset --hard origin/main && chmod +x setup && ./setup
```

## Desinstalação

```
for s in hm-init hm-engineer hm-designer hm-qa hm-deploy; do rm -f ~/.claude/skills/$s; done && rm -rf ~/.claude/skills/highermind-code-skills
```

---

## Filosofia

Construído em cima da filosofia [Higher Mind](https://highermind.com.br):

**Primeiro o fundador. Depois a empresa. Depois o mundo.**

O mesmo vale pro software. Primeiro o padrão. Depois o código. Depois o produto. Se o padrão é world-class, tudo que vem depois também será.

Empresas são só o subproduto. Software é só o subproduto. O que importa é a mente que decidiu construir.

---

## Licença

MIT

---

Criado por [Rodrigo Lopes](https://github.com/rodrigohighermind), fundador do [Higher Mind](https://highermind.com.br).
