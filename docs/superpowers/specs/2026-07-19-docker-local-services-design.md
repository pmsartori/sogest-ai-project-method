# Sogest AI Project Method — serviços locais via Docker (tipo Software)

**Status:** accepted
**Date:** 2026-07-19

## Contexto

O template de Software não oferece nenhuma forma padronizada de rodar
serviços locais (banco de dados, cache) — cada projeto resolve isso do
próprio jeito. O caso real que motivou isto: o Prisma CFO precisa de
Postgres local, e esse ambiente foi montado manualmente (`docker run
--name prisma-pg-local ...`), sem nada versionado no repositório — uma
pessoa nova precisa redescobrir os comandos exatos sozinha, em vez de só
rodar `docker compose up`.

Um Docker Compose versionado no template resolve isso para os próximos
projetos, sem forçar a dependência em projetos que não precisam de nenhum
serviço local (ex.: um frontend puro).

## Decisão

Adicionar um bloco **opcional** ao template de Software, seguindo o mesmo
padrão já usado pelas variantes de deploy (pasta descartável, aplicada
condicionalmente, sempre removida no final da Fase 2).

### Estrutura nova no template (`sogest-project-template`)

```
sogest-project-template/
└── optional/
    ├── docker-compose.yml                        [SLUG_PROJETO] no nome do container/banco
    └── CONTRIBUTING-local-services-snippet.md     trecho "## Ambiente local" a anexar
```

`docker-compose.yml` sobe **só um serviço Postgres**, como exemplo — cobre o
caso mais comum (é o que o Prisma usa hoje) sem impor uma stack completa:

```yaml
services:
  db:
    image: postgres:16
    container_name: [SLUG_PROJETO]-pg-local
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: localdev
      POSTGRES_DB: [SLUG_PROJETO]
    ports:
      - "5432:5432"
    volumes:
      - [SLUG_PROJETO]_pg_data:/var/lib/postgresql/data
# Adicione outros serviços (Redis, etc.) aqui conforme o projeto precisar.
volumes:
  [SLUG_PROJETO]_pg_data:
```

`CONTRIBUTING-local-services-snippet.md` documenta como subir/derrubar o
ambiente e onde ajustar credenciais — vira uma seção `## Ambiente local` no
`CONTRIBUTING.md` do projeto gerado, só quando aplicável.

### Fluxo da skill (`skills/new-project/SKILL.md`)

**Fase 1** ganha uma pergunta nova, após a pergunta de módulos e antes do
modelo de deploy:

> Este projeto precisa de serviços locais (banco de dados, cache)? (sim/não)

- Se **não**: nada muda — a Fase 2 segue exatamente como hoje.
- Se **sim**: antes de avançar, a skill checa `docker info` (Docker
  instalado e rodando). Se falhar, instrui a instalar/abrir o Docker
  Desktop e **pausa** — mesma disciplina das checagens de pré-voo já
  existentes (nunca prossegue com uma dependência ausente sem avisar).

**Fase 2** ganha um passo condicional, entre "aplicar modelo de deploy" e
"commit em main":

- Se a resposta foi **sim**: copia `optional/docker-compose.yml` para a
  raiz do projeto (substituindo `[SLUG_PROJETO]`), e anexa o conteúdo de
  `optional/CONTRIBUTING-local-services-snippet.md` ao final do
  `CONTRIBUTING.md` já preenchido.
- **Em qualquer um dos dois casos** (sim ou não), a pasta `optional/` é
  apagada antes do commit — mesma disciplina já aplicada a `variants/`, para
  que nada de descartável sobre no repositório final.
- A skill **nunca roda `docker compose up` sozinha**. O resumo final (Fase
  2, último passo) passa a incluir, quando aplicável, a instrução `docker
  compose up -d` como próximo passo manual.

## Consequências

- Cobre só o caso Postgres — se um projeto precisar de outro serviço
  (Redis, RabbitMQ etc.), a pessoa edita o `docker-compose.yml` gerado à
  mão; o arquivo já tem um comentário indicando onde adicionar.
- Fora do escopo: nenhuma tentativa de detectar automaticamente se um
  projeto "precisa" de banco — a pergunta é sempre feita explicitamente na
  Fase 1, a decisão fica com a pessoa.
- Sem impacto no tipo "Metodologia e Gestão" (ainda não implementado) — essa
  pergunta só existe no fluxo do tipo Software.
- Credenciais (`dev`/`localdev`) são fixas e claramente marcadas como
  ambiente local, não segredo real — consistente com o `.env.example` que o
  próprio Prisma já usa hoje para o mesmo propósito.
