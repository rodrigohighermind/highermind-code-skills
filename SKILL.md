# highermind-code-skills (v3 — Security-First)

Cinco modos cognitivos de execucao pro Claude Code, construidos na filosofia Higher Mind. **Seguranca e pre-requisito bloqueante em todas as skills, nao checklist final.**

## Skills disponiveis

- `/hm-init` — Comecar um novo projeto com as melhores ferramentas, estrutura e seguranca desde o commit zero
- `/hm-engineer` — Validar codigo em todas as camadas (seguranca OWASP + container primeiro, depois arquitetura, performance, custo, qualidade)
- `/hm-designer` — Validar interface contra o mais alto padrao de design de software
- `/hm-qa` — Security audit + testes + verificacao completa
- `/hm-deploy` — Security gate bloqueante + validacao de infraestrutura, containers e reprodutibilidade

Skills de direcao estrategica (`/hm-align`, `/hm-sequoia`) estao em [highermind-business-skills](https://github.com/rodrigohighermind/highermind-business-skills).

Cada skill tem seu proprio SKILL.md na subpasta.

## Principio v3

Na v2, seguranca era uma camada de auditoria. Na v3, e gate. A diferenca: se `.dockerignore` nao existe, se o Dockerfile roda dev server, se secrets estao expostos — o veredicto e BLOQUEADO. Nao importa se tudo mais funciona.

## CLAUDE.md

O `CLAUDE.md.template` incluido fornece um arquivo de identidade global que define o padrao world-class como contexto always-on pra toda sessao.

Construido na filosofia Higher Mind: primeiro a seguranca, depois o padrao, depois o codigo, depois o produto.
