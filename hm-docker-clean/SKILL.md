---
name: hm-docker-clean
description: Faxina segura de Docker no padrão Higher Mind (dados sagrados). Use quando o disco do Mac está cheio por causa do Docker, quando o Docker.raw inchou, ou quando o Owner pede pra limpar imagens/cache/containers. Codifica o protocolo de limpeza profunda ASSISTIDA — para antes de tocar qualquer coisa de produto, verifica onde o dado realmente vive, nunca apaga volume. Para o lixo trivial recorrente existe o binário `docker-gc` (cache+dangling, conservador, agendado no launchd).
---

# /hm-docker-clean — Faxina de Docker Segura

Você está em **modo limpeza de Docker**. Regra mãe: **dados são sagrados**. Disco cheio nunca justifica perder um byte de banco.

## Antes de tudo: existe automação pro lixo trivial

O binário **`docker-gc`** já cuida do lixo recorrente e seguro (build cache órfão + imagens dangling), conservador, agendado no launchd semanal (`com.highermind.docker-gc`). Se o problema é só acúmulo de rotina, rode `docker-gc` e pare aqui. Esta skill é pro caso que **exige julgamento**: imagens grandes de produtos, decidir o que pode sair.

## Proibições absolutas (nunca, em hipótese alguma)

- `docker volume prune` — destrói dados de produtos.
- `docker system prune -a --volumes` — idem, pior.
- `docker volume rm <qualquer>` sem confirmação explícita do Owner naquela sessão.
- Remover imagem com tag de produto sem antes provar onde o dado vive.

## Protocolo (a ordem importa)

### 1. Diagnóstico, sem tocar em nada
```
docker system df -v      # o que ocupa: imagens, cache, containers, VOLUMES
docker volume ls         # liste e confirme com o Owner quais são os bancos dele
```
Mapeie cada volume ao produto. Os volumes são o ativo sagrado — eles **não saem nunca** nesta faxina.

### 2. Limpe só o descartável puro (sem perguntar)
```
docker builder prune -f  # cache órfão
docker image prune -f    # SÓ dangling (<none> sem container). Nunca -a sem confirmar.
```

### 3. PARE antes de containers e imagens de produto
- `docker container prune -f` remove containers parados. **Não apaga volume** (o dado fica no volume nomeado, reconecta no `compose up`). Mas todos costumam ser produtos do Owner → **confirme antes**, listando exatamente o que sai.
- Imagens com tag de produto → **prove onde o dado vive antes de remover**. Faces/pessoas, transações, etc. costumam estar em **Supabase/Postgres (volume ou nuvem)**, não na imagem. A imagem é só o ambiente de runtime/build. Remover a imagem ≠ perder dado — mas **só afirme isso depois de checar** (grep no projeto por Dockerfile/compose que a referencie; ver se a tabela vive em volume local ou Supabase).
- Custo real de remover imagem de produto = ter que **rebuildar** depois. Diga isso ao Owner.

### 4. Cache pesado (`builder prune -a`)
Libera o cache de builds vivos também. Custo: próximo build de tudo fica lento (do zero). Vale quando o cache está em muitos GB e é morto. Confirme o trade-off.

### 5. O Docker.raw não encolhe sozinho
Apagar imagem libera espaço **dentro** do `Docker.raw`, mas o arquivo (no `~/Library/Containers/com.docker.docker/...`) não auto-compacta. Pra devolver espaço ao disco do Mac:
> **Quit no Docker Desktop e reabrir**, ou **Settings → Resources → Reclaim space**.
Esse passo é do Owner (mexe no app inteiro). Só vale **depois** de remover as imagens grandes.

### 6. Balanço final
```
docker system df         # mostre o liberado
```
Confirme: contagem de volumes igual à do passo 1 (nenhum perdido), containers de produto vivo ainda UP.

## Padrão de comunicação nesta skill

- Sempre separar "espaço liberado dentro do Docker" de "espaço devolvido ao disco" (só o reclaim faz o segundo).
- Antes de remover qualquer coisa de produto: listar exatamente o que sai, o custo (rebuild), e onde o dado vive.
- Nunca rodar operação destrutiva de volume. Se o Owner pedir, confirmar duas vezes e fazer backup antes.
