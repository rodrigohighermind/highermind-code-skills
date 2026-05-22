---
name: hm-data-integrity
description: Dados sagrados — backup, migration safety, operações destrutivas, runtime integrity, schema validation, DR plan, compliance, integridade de arquivos. Use antes de shippar projeto que persiste qualquer coisa (DB, files, blob), após mudança em schema/migration, ao definir política de backup, após incidente envolvendo perda de dados, periodicamente em projetos com dados pessoais/financeiros. Cobre nível "produção pessoal" (Electron/Tauri com DB em userData) explicitamente.
---

# /hm-data-integrity — Dados Sagrados (v2)

Você está agora em **modo data integrity**. Seu trabalho e validar que dados do user não sao perdidos. Nem por bug, nem por crash, nem por migration ruim, nem por operação destrutiva acidental, nem por disco morto. Dados sao sagrados — esse princípio precisa de implementação concreta, não só promessa.

## Princípio central

A única perda aceitável de dado e a explicitamente autorizada pelo owner. Tudo o mais é bug. Se dados podem ser perdidos por um comando errado, um crash mal tratado, ou uma migration apressada, o produto não está pronto pra ter usuarios — nem ele mesmo.

## Quando usar

- Antes de shippar projeto que persiste qualquer coisa (DB, files, blob storage)
- Apos mudanca em schema ou migration
- Quando definir política de backup
- Apos qualquer incidente envolvendo perda de dados
- Periodicamente (auditoria de manutencao, especialmente projetos com dados pessoais/financeiros)

## Níveis

| Nível | Quando | Foco |
|---|---|---|
| **Local single-user** | Apps pessoais em dev (CLI local, ferramenta ainda não instalada) | Backup local automático, migration safety, undo de destrutivos |
| **Produção pessoal** | App pessoal instalado (`.app` em /Applications, Tauri/Electron com DB em `userData`, single-user em uso real) | Tudo de local + tratar TODO destrutivo como prod. Sem reset de profile/facts/DB sem confirmacao explícita do user por operação |
| **Multi-user privado** | CRM interno, ferramentas de time | Tudo acima + replicacao, RTO/RPO, audit trail |
| **Produção pública** | SaaS com clientes pagantes | Tudo acima + DR plan, geo-redundancia, compliance (LGPD/GDPR) |

**Produção pessoal e armadilha clássica.** Código roda numa máquina só e parece "dev", mas o DB carrega histórico real e irrecuperavel (notas pessoais, fatos, decisões registradas pelo próprio user). Tratar como prod desde o primeiro `npm run tauri build` / `electron-builder` que vai pra /Applications ou `Program Files`. Regra: qualquer DELETE, sobrescrever DB, reset de profile/facts ou migration não-trivial exige confirmacao explícita do user por operação, mesmo na máquina do dev.

## Dominios

### 1. Backup strategy

| Check | Criterio |
|---|---|
| Backup automático | Existe? Quando dispara (schedule, evento)? |
| Backup atômico | Snapshot consistente, não corrompido por write em curso? (ex: sqlite `.backup`, pg_dump --serializable) |
| Backup criptografado | Em repouso e em transito (se vai pra remoto) |
| Backup versionado | Multiplas gerações. Não só o "último" — histórico |
| Backup testado | Você já restaurou um backup com sucesso? Se não, não tem backup, tem **esperança**. |
| Backup off-site | Pra projetos importantes: copia geografica separada (S3 outra regiao, git remoto, etc) |
| Retention policy | Daily 7d, weekly 4w, monthly 12m? Definido? |

**Patterns:**
- **Single-user app local:** snapshot SQLite no `before-quit` do Electron + commit/push pra repo privado
- **Multi-user:** wal-g pg backups continuous + S3 versioned bucket
- **Files:** rclone sync incremental encrypted

### 2. Migration safety

| Check | Criterio |
|---|---|
| Migrations versionadas | Drizzle, Prisma, Alembic — sequencial, hash-locked |
| Migrations idempotentes | Re-rodar não quebra (auto-migrate na boot) |
| Migrations roll-forward only? | Production = sim. Dev/local = pode roll-back |
| Migration destrutiva exige confirmacao | DROP COLUMN, DROP TABLE — never silent |
| Backup ANTES de migration grande | Auto ou manual? Documentado? |
| Migration testada em copia de prod | Antes de aplicar em prod real |
| Rollback plan documentado | Mesmo que roll-forward only, plano se algo der errado |
| Schema journal sincronizado | Drizzle/Prisma journal bate com estado real do DB? |

**Anti-patterns:**
- ALTER COLUMN sem default/backfill (NOT NULL em coluna nova com rows existentes)
- DROP COLUMN sem etapa intermediaria de read-only
- Migration que demora muito sem `CONCURRENTLY` em indexes (Postgres lock)
- Schema mudou no código sem migration correspondente

### 3. Operações destrutivas

**Lista canonica de operações que EXIGEM confirmacao explícita do owner** (cravada no `~/.claude/CLAUDE.md` global, autorizacao vale apenas pra sessão atual e operação especifica):

- `docker compose down -v` (apaga volumes nomeados — destroi DB)
- `rm -rf` em path não temporario
- `git push --force` em qualquer branch (especial cuidado em main/master)
- `git reset --hard` em branch já publicada
- `git branch -D` ou delete de branch publicada no remote
- `DROP TABLE`, `DROP DATABASE`, `DROP SCHEMA`
- `TRUNCATE` em tabela com dados
- `DELETE` sem `WHERE` especifico
- Mass `UPDATE` sem WHERE
- Kill de processo de produção
- Reset/sobrescrever DB em produção pessoal (ex: SQLite em `userData`/`AppData`)
- Reset de profile/facts/memoria persistente em app pessoal em uso
- Restore de backup que sobrescreve estado atual sem snapshot do atual

**A única exceção:** owner já autorizou EXPLICITAMENTE a operação especifica na sessão atual. Autorizacao não retroage e não se estende a operações parecidas.

| Check | Criterio |
|---|---|
| DELETE sem WHERE bloqueado | ORM raw SQL não executa "DELETE FROM x" sem clause |
| Soft-delete por default | `deleted_at` ao inves de DROP. Hard-delete e exceção auditada |
| Confirmacao explícita pra hard-delete | UI: dialog com nome do objeto. CLI: --force flag |
| Backup automático antes de bulk operation | DROP TABLE, TRUNCATE, mass UPDATE — backup primeiro |
| Audit log de destrutivos | Quem fez, quando, o que removeu |
| Recovery window | Soft-delete fica X dias antes de purge real |
| Owner approval pra ops em prod | Migration prod, mass delete, schema change — owner em loop |
| Produção pessoal tratada como prod | App instalado/em uso com DB em userData = prod. Sem reset sem confirmacao |

**Pattern de confirmacao (CLI):**
```bash
$ tool delete user 123
ERROR: Destructive operation. Add --force to confirm.

$ tool delete user 123 --force
Backing up affected rows to /tmp/backup-...
Deleted 1 user, 47 related records.
```

### 4. Data integrity em runtime

| Check | Criterio |
|---|---|
| Transactions onde atomicidade importa | Multi-table writes em transaction |
| Foreign keys + ON DELETE CASCADE configurado | Sem orphaned records |
| Unique constraints em campos que devem ser únicos | DB enforced, não só app-level |
| NOT NULL em campos obrigatorios | DB enforced |
| Check constraints pra invariantes | `amount >= 0`, `email LIKE '%@%'`, etc |
| Optimistic concurrency em writes simultaneos | `updated_at`/`version` no WHERE de UPDATE |
| Idempotency keys em operações externamente disparadas | Webhook, payment, email send |

### 5. Schema validation runtime (JSON.parse + cast)

| Check | Criterio |
|---|---|
| JSON column tem schema versionado | `payload.schemaVersion` field |
| safeParse via Zod/Pydantic em todo JSON.parse de DB | Sem cast cego |
| Migration de schema versionado documentada | Como fazer v1 → v2 sem perder dados |
| Backwards compat enquanto migra | App lida com v1 E v2 durante transicao |

### 6. Disaster recovery (multi-user / produção)

| Métrica | Definicao | Típico |
|---|---|---|
| **RPO** (Recovery Point Objective) | Quanto dado pode perder? | 5min a 1h dependendo da criticidade |
| **RTO** (Recovery Time Objective) | Quanto tempo pra voltar? | 15min a 4h |
| **Disaster scenarios documentados** | DB corrompido, regiao AWS down, ransomware, dev errou comando | Pelo menos 4 cenarios cobertos |
| **DR drill executado** | Restore real testado em ambiente paralelo | Ao menos 1x/ano |
| **Runbook de recuperacao** | Passo-a-passo escrito | Acessivel mesmo se app down |

### 7. Compliance (se aplicavel — LGPD/GDPR/HIPAA)

| Check | Criterio |
|---|---|
| Right to erasure | User pode pedir exclusao real? Dados removidos de backups apos retention? |
| Right to access | User pode exportar TODOS seus dados em formato legivel? |
| Audit log de acessos | Quem visualizou dados sensíveis (PII, saúde, financeiro)? |
| Breach notification plan | Procedimento de 72h pra ANPD/DPA documentado |
| DPO designado | (LGPD/GDPR) |
| Data minimization | Coleta mínima pro proposito declarado |

### 8. File/blob integrity (se app maneja arquivos)

| Check | Criterio |
|---|---|
| Checksum em uploads | SHA256 calculado e verificado |
| Versioning em blob storage | S3 versioned bucket pra recovery de overwrite acidental |
| Lifecycle policy | Auto-archive pra cold storage apos N dias |
| Object lock em arquivos criticos | Imutabilidade WORM pra compliance/legal |
| Backup de blob storage | S3 cross-region replication ou snapshot externo |

### 9. Observabilidade pra detectar problemas cedo

- Alerta quando backup falha
- Alerta quando DB size cresce muito rapido (potencial bug ou ataque)
- Alerta quando query lenta (indica index missing)
- Alerta quando rate de DELETE alto (indica bug ou ataque)
- Dashboard com volume de dados, último backup, tamanho dos backups

## Output

```
DATA INTEGRITY AUDIT
Projeto: [nome]
Nível aplicado: [Local / Multi-user / Produção pública]
Volume estimado: [rows/files, tamanho]

DOMÍNIO 1: BACKUP
[Check]: PASS/FAIL — detalhes + fix se FAIL

DOMÍNIO 2: MIGRATION
[Check]: PASS/FAIL

DOMÍNIO 3: DESTRUTIVAS
[Check]: PASS/FAIL

DOMÍNIO 4: RUNTIME INTEGRITY
[Check]: PASS/FAIL

DOMÍNIO 5: SCHEMA VALIDATION
[Check]: PASS/FAIL

DOMÍNIO 6: DR (se Multi-user/Produção)
RPO atual: [valor]
RTO atual: [valor]
Drill executado: [data último / nunca]

DOMÍNIO 7: COMPLIANCE (se aplicavel)
[Check]: PASS/FAIL

DOMÍNIO 8: FILES (se aplicavel)
[Check]: PASS/FAIL

DOMÍNIO 9: OBSERVABILIDADE
[Check]: PASS/FAIL

VEREDICTO
Dados protegidos / EM RISCO — X criticos pra resolver primeiro
```

## Regras

- **Toda perda potencial de dado é CRÍTICO.** Sem MEDIO. Sem BAIXO.
- Backup que nunca foi restaurado não é backup. Se nunca testou, marca FAIL.
- Migration sem rollback plan é CRÍTICO em produção.
- DELETE sem confirmacao em CLI/UI público = CRÍTICO.
- Disco morto e cenario OBRIGATÓRIO em DR plan.
- Owner aprova política de retention. Default conservador: não apagar nada que pode ser útil.
- Se projeto e single-user local: backup automático no quit + commit pra repo privado e baseline mínimo.
- **Produção pessoal não é dev.** App instalado em /Applications + DB em `userData` (Tauri/Electron) = produção. Confirmacao explícita por operação destrutiva, sem exceção.
- Compliance falha = não shippa pra mercado regulado. Sem negociacao.
