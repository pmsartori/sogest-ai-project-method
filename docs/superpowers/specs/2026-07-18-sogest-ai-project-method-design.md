# Sogest AI Project Method — design

**Status:** approved
**Date:** 2026-07-18

## Context

O repositório `waltmonaco-ux/prisma-cfo` acumulou, ao longo do projeto, uma
forma de organizar trabalho que funciona bem: módulos → epics → tasks →
sub-tasks no GitHub, ADRs para decisões, um método fixo
(brainstorm→spec→plan→código→review) apoiado em skills do Claude Code, e um
pipeline de deploy em 3 ambientes. Isso hoje só existe *dentro* do Prisma —
não há como replicar essa estrutura para um projeto novo sem reconstruir tudo
manualmente, nem como outra pessoa (fora da máquina do Pedro) usar o mesmo
método sem copiar arquivos à mão.

O objetivo deste projeto é extrair esse padrão para algo instalável e
reutilizável — um "instalador" de gestão de projetos com IA + GitHub — que
qualquer pessoa com GitHub e Claude Code consiga rodar para começar um
projeto novo já organizado, sem depender do Prisma como referência e sem
depender do Pedro para configurar cada pessoa manualmente.

## Decisão

Construir **dois repositórios GitHub públicos, sob `pmsartori`**, com papéis
diferentes e sem sobreposição:

### 1. `pmsartori/sogest-project-template` — GitHub Template Repository

Contém *apenas* os arquivos que devem existir dentro de um projeto novo.
Marcado como "Template repository" nas configurações do GitHub, o que permite
`gh repo create --template pmsartori/sogest-project-template` copiar toda a
árvore instantaneamente para o repositório novo.

```
sogest-project-template/
├── AGENTS.md                     placeholders: [PROJETO], [MODULO_1..N]
├── CONTRIBUTING.md                inclui tabela de ambientes (condicional ao deploy escolhido)
├── README.md
├── docs/
│   ├── PROJECT.md                 ★ o manual humano — meta-modelo genérico com placeholders
│   ├── glossary.md                esqueleto vazio + instruções de preenchimento
│   ├── DEPLOYMENT.md              conteúdo varia conforme o modelo de deploy
│   ├── WORKLIST.md
│   ├── how-we-build-a-module.md   método brainstorm→spec→plan→code, genérico
│   └── adr/
│       └── README.md              formato do ADR + regra "suceder, nunca editar"
├── .github/
│   ├── ISSUE_TEMPLATE/{bug_report.yml, suggestion.yml, config.yml}
│   └── workflows/
│       ├── ci.yml                 lint/test/build — sempre incluso
│       └── variants/               NÃO permanece no projeto final
│           ├── deploy-ssh/         deploy-dev.yml, deploy-test.yml, deploy-prod.yml
│           ├── deploy-vercel/      integração via GitHub App da Vercel + regras de branch
│           └── deploy-manual/      só documentação de promoção, sem workflow
└── scripts/
    ├── overview.sh                 genérico — lê módulos dinamicamente via labels area:*
    └── setup-board.sh              parametrizado por OWNER/REPO/TITLE via variáveis
```

### 2. `pmsartori/sogest-ai-project-method` — Claude Code Plugin (marketplace)

Contém a skill interativa (o "instalador") e as referências de cada modelo de
deploy. **Não contém arquivos de projeto** — sabe apontar para o template
acima. Distribuído do mesmo jeito que o Superpowers: a pessoa roda
`/plugin marketplace add pmsartori/sogest-ai-project-method` e depois
`/plugin install`.

```
sogest-ai-project-method/
├── .claude-plugin/
│   └── marketplace.json
└── sogest-ai-project-method/
    ├── plugin.json
    └── skills/
        └── new-project/
            ├── SKILL.md
            └── references/
                ├── deploy-ssh.md
                ├── deploy-vercel.md
                └── deploy-manual.md
```

> A forma exata do manifest do plugin (`marketplace.json`/`plugin.json`) será
> verificada contra a documentação oficial de plugins do Claude Code durante
> a implementação — este design fixa a estrutura conceitual, não o schema
> byte-a-byte.

## Fluxo da skill (`new-project`)

### Fase 0 — Pré-voo

Executado antes de qualquer pergunta de negócio:

| Checagem | Se falhar |
|---|---|
| `gh` instalado | Instrui instalação (link oficial), pausa o fluxo |
| `gh auth status` autenticado | Instrui `gh auth login`, pausa (login é inerentemente interativo) |
| Escopo `project` presente | Instrui `gh auth refresh -s project,read:project` |
| Plugin `superpowers` instalado | Oferece instalar a partir da marketplace oficial (`claude-plugins-official`) |
| Plugin `claude-mem` instalado | Oferece instalar a partir da marketplace oficial (`thedotmack`) |

Os plugins de dependência **nunca são copiados/vendorizados** — apenas
checados e, se ausentes, instalados a partir das marketplaces oficiais deles.
Isso evita cópias congeladas que ficam desatualizadas; quem já tem os
plugins instalados só é notificado, não reinstala.

**Por que `claude-mem` também é dependência (e não só `superpowers`):**
o método (brainstorm→spec→plan→code) documenta o *porquê* das decisões, mas
não resolve coordenação entre múltiplas pessoas/sessões no mesmo repositório
ao longo do tempo. `claude-mem` cobre esse eixo especificamente:
`mem-search` (evita retrabalho — "já resolvemos isso?"), `standup`
(consolidação read-only entre worktrees/branches/PRs), `oh-my-issues`
(triagem de backlog grande), `babysit` (acompanhar PR até mergeable),
`learn-codebase` (onboarding rápido de gente nova). Um projeto pensado para
mais de uma pessoa precisa dos dois eixos — método e coordenação — não só do
primeiro.

### Fase 1 — Perguntas

1. Nome do projeto (vira slug do repo) + descrição de uma linha
2. Privado ou público
3. Módulos do projeto (lista livre, ex: "Tenancy, Reporting, Import") — cada
   um vira uma label `area:*` + 1 epic
4. Modelo de deploy: **SSH próprio** / **Vercel-Netlify** / **Nenhum
   (manual)**
   - se SSH: host, usuário, caminho remoto (credenciais reais ficam como
     próximo passo manual, por segurança)
5. Cadência de sprint (ex: 2 semanas) → cria o milestone "Sprint 1"

### Fase 2 — Automação

1. `gh repo create <slug> --template pmsartori/sogest-project-template
   --public|--private`
2. Clona local, substitui todos os placeholders pelos valores respondidos
3. Move os workflows do modelo de deploy escolhido para `.github/workflows/`,
   apaga a pasta `variants/`
4. Cria as labels: `area:*` dinâmicas (uma por módulo) + conjunto fixo
   (`epic`, `status:ready|in-progress|in-review|blocked`, `needs-triage`,
   `accepted`, `suggestion`, `good first issue`, `tech-debt`, `security`)
5. Cria 1 issue `[EPIC]` por módulo, label `epic` + `area:<módulo>`
6. Cria o milestone da Sprint 1, com data de vencimento calculada pela
   cadência informada
7. Cria o Project board (colunas Triage→Backlog→Ready→In Progress→In
   Review→Done) e liga os automatismos nativos (item novo → Triage; PR
   merged → Done) — reaproveitando a lógica de `scripts/setup-board.sh`
8. Cria as branches `develop` e `test` a partir de `main`; protege `main`
   (PR + review obrigatórios)
9. Commit `chore: fill onboarding placeholders` + push
10. Imprime resumo: link do repo, do board, dos milestones, e a lista de
    próximos passos manuais (ex.: configurar secrets de deploy
    `SSH_HOST`/`SSH_KEY` se o modelo escolhido foi SSH)

## O manual humano (`docs/PROJECT.md`)

Vive no template, escrito de forma genérica (meta-modelo, com
`[placeholders]`), cobrindo as 4 camadas descritas nas conversas de design:

1. **Código** — como o repositório é organizado (docs/, scripts/, ADRs)
2. **Git flow** — branches, promoção entre ambientes, quem pode mexer onde
3. **Tracking** — módulo → epic → task → sub-task, labels, board, sprints
4. **Método** — brainstorm → spec → plan → code+test → review

A Fase 2 da skill preenche os placeholders com as respostas do onboarding,
então cada projeto novo já nasce com seu próprio manual, específico dele, sem
o Pedro precisar escrever nada à mão depois.

## Consequências

- Dois repositórios para manter em vez de um; separação evita que metadados
  de plugin vazem para dentro de projetos gerados, e que arquivos de projeto
  poluam a marketplace do plugin.
- Repositórios públicos: qualquer pessoa com `gh` consegue usar o template
  sem depender de convite manual do Pedro; em troca, a estrutura/documentação
  do método fica publicamente visível (não há segredo nela).
- Dependências de plugin (`superpowers`, `claude-mem`) são checadas, não
  vendorizadas — atualizações futuras desses plugins chegam automaticamente
  a quem já instalou, sem exigir nova versão do `sogest-ai-project-method`.
- Infra de deploy real (credenciais SSH, tokens Vercel) permanece fora do
  escopo automatizável — a skill configura a estrutura e documenta o
  próximo passo manual, mas nunca gera/injeta segredos sozinha.
- Escopo deste design é a criação do projeto novo. Não cobre: sincronização
  contínua entre o template e projetos já criados a partir dele (um projeto
  gerado não "puxa" atualizações futuras do template automaticamente).
