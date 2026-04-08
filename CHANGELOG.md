# Changelog

Todas as mudanças notáveis neste projeto serão documentadas aqui.

Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/).

---

## [3.0.0] — 2026-04-08

Security-first evolution. Todas as skills agora tratam seguranca como pre-requisito bloqueante, nao checklist final. Motivado por falhas reais encontradas no Orion Finance (Dockerfile rodando `npm run dev` em producao, sem `.dockerignore`, `--reload` no entrypoint) que passaram pelas skills v2 sem ser detectadas.

### Adicionado

#### `/hm-deploy` — Security Gate bloqueante
- Nova secao **0. Security Gate** que roda ANTES de tudo
- Se qualquer item CRITICO falha, o deploy e BLOQUEADO automaticamente
- Tabela completa de checks com severidades explicitas
- `.dockerignore` ausente = PARA TUDO, cria antes de continuar
- Dev server em producao (`npm run dev`, `--reload`) = CRITICO
- Non-root user, build secrets, CORS, Swagger/debug — tudo com severidade definida
- Output atualizado com secao SECURITY GATE no topo
- Tabela de ports atualizada com todos os projetos (incluindo orion-finance e hm-product-lab)

#### `/hm-engineer` — OWASP Top 10 e Container Security
- Nova secao **Seguranca — Aplicacao (OWASP Top 10)** com tabela completa dos 10 items
- Nova secao **Seguranca — Container & Infraestrutura** com 9 checks explicitos e severidades
- Nova secao **Seguranca — Dependencias** (CVEs, lock files, supply chain)
- Regra: seguranca e SEMPRE a primeira camada auditada
- Regra: se `.dockerignore` nao existe, parar auditoria ate resolver
- Regra: todo finding de seguranca e CRITICO, sem excecao

#### `/hm-init` — Seguranca desde o primeiro commit
- Nova secao **6. Seguranca desde o primeiro commit** (OBRIGATORIA)
- `.dockerignore` template completo — obrigatorio em todo projeto com Docker
- Dockerfiles production-ready com exemplos (Python multi-stage, Next.js standalone)
- Entrypoints separados dev/prod como padrao
- Regras de secrets reforçadas
- Dependency audit no init
- Health check que verifica dependencias reais
- Criterio de **Seguranca** adicionado a tabela de avaliacao de stack

#### `/hm-qa` — Security Audit como primeira etapa
- Nova secao **0. Security Audit** que roda ANTES de testes funcionais
- Sub-secoes: Dependencias, Container Security, OWASP Quick Scan, Secrets Scan
- Grep por patterns de secrets no codebase
- Verificacao de secrets em commits anteriores (`git log --all -S`)
- Prioridade de testes reordenada: seguranca primeiro
- Finding de seguranca bloqueia ship

### Mudado

- Todas as 4 code skills agora marcadas como v2 no titulo
- Seguranca movida de "uma das camadas" para "pre-requisito bloqueante"
- Output de todas as skills agora tem secao de seguranca no topo
- Severidades de seguranca padronizadas: CRITICO (nunca MEDIO, nunca BAIXO)

### Filosofia

A v2 tratava seguranca como uma camada de auditoria. A v3 trata como gate. A diferenca: na v2, voce podia listar um problema de seguranca como MEDIO e continuar. Na v3, seguranca bloqueia. Nao importa se o resto funciona perfeitamente — se `.dockerignore` nao existe, se o Dockerfile roda dev server, se secrets estao expostos, o veredicto e BLOQUEADO.

---

## [2.0.0] — 2026-04-03

Evolução completa das skills baseada em aprendizados reais de 5+ projetos construídos com o framework (higher-mind-os, Scout, HM Finance, Orion, higher-mind-community).

### Adicionado

#### Nova skill: `/hm-deploy`
- Validação completa de infraestrutura, containers e reprodutibilidade
- Checklist de Docker (subida, rebuild, dados sagrados)
- Validação de environment e secrets
- Checklist de database e migrations
- Health checks e monitoramento
- Teste de reprodutibilidade (clone limpo)
- Segurança de deploy (ports, CORS, HTTPS, secrets)
- Tabela de ports dos projetos Higher Mind pra evitar colisões

#### `/hm-init` — Framework de decisão de stack
- Tabela de critérios ponderados pra avaliação de stack (fit, performance, custo, maturidade, ecossistema, DX, hiring)
- Anti-patterns de escolha explícitos
- Seção de arquitetura agent-first como default (quando aplicável)
- Infraestrutura local com Docker Compose desde o dia 1
- Restrições de custo como parte do design (API calls, hosting, bandwidth)
- Documentação obrigatória via ARCHITECTURE.md
- Princípio "dados são sagrados" desde o primeiro docker-compose.yml

#### `/hm-engineer` — Padrão senior e novas camadas de auditoria
- Baseline de engenheiro senior inegociável (zero bare except, zero any types, zero fire-and-forget, zero secrets hardcoded, zero queries sem limit)
- Nova camada: **Custo x Performance** (API calls justificadas, contexto mínimo em LLMs, token usage consciente)
- Nova camada: **Dados sagrados** (nenhuma operação destrutiva sem confirmação, volumes nomeados, migrations não-destrutivas)
- Nova camada: **Infraestrutura** (Docker rebuild vs restart, migrations automáticas, health checks, ports)
- Expansão de Performance: I/O paralelo, memoização
- Expansão de Arquitetura: validação de agent loops (max iterations, token limits, timeout)
- Regra: dados em risco é sempre CRÍTICO

#### `/hm-designer` — Agent-first UI e pixel perfect
- Filosofia agent-first: UI = visibilidade + override, não input principal
- Referência A24 adicionada às referências estéticas
- Seção **Pixel perfect**: zero tolerância a desalinhamentos, quebras, cortes
- Padrões técnicos: full-width layout, shimmer (não spinner), dark-first, inline styles quando framework não coopera, transições 200-300ms
- Novos anti-patterns: formulários onde agente deveria executar, spinners genéricos, layout centralizado em telas grandes
- Regra: desalinhamento arquitetural (formulário vs agente) é finding

#### `/hm-qa` — Infraestrutura, agente e custo como teste
- Nova seção: **Verificação de infraestrutura** (containers, migrations, ports, volumes, rebuild, .env)
- Nova seção: **Verificação de agente** (tool loops, alucinação de tools, token usage, custo por interação)
- Nova seção: **Integridade de dados** (persistência, migrations não-destrutivas, backups, operações destrutivas)
- Nova seção: **Check de custo** (API calls por fluxo, contexto mínimo, calls redundantes, custo por usuário/mês)
- Output expandido com seções de Infraestrutura, Agente, Integridade de Dados e Custo

### Mudado

- Contagem de skills: 4 → 5 (adição de `/hm-deploy`)
- SKILL.md parent atualizado pra refletir 5 skills
- Todas as skills agora consideram agent-first como paradigma (quando aplicável)
- Performance expandida de "não ter N+1" pra incluir custo de APIs externas e token management

---

## [1.0.0] — 2026-03-12

Release inicial.

### Skills
- `/hm-init` — Início de projeto com melhores ferramentas e estrutura
- `/hm-engineer` — Validação de código em todas as camadas
- `/hm-designer` — Validação de interface contra o mais alto padrão
- `/hm-qa` — Quality assurance completo

### Infraestrutura
- Setup script com symlinks automáticos
- CLAUDE.md.template como ponto de partida
- Instalação global e por projeto
