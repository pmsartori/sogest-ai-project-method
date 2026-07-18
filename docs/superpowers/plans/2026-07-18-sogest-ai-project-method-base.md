# Sogest AI Project Method — Base Installer (tipo Software) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Construir o instalador `sogest-ai-project-method` (Claude Code Plugin) e o repositório-template `sogest-project-template`, cobrindo o tipo de projeto "Software" ponta a ponta: da pergunta inicial até um repositório GitHub novo, organizado, no ar.

**Architecture:** Dois repositórios GitHub públicos sob `pmsartori`. `sogest-project-template` é um GitHub Template Repository puro (só arquivos de projeto). `sogest-ai-project-method` é simultaneamente a marketplace e o plugin (mesmo padrão usado pelo próprio Superpowers: `.claude-plugin/marketplace.json` + `.claude-plugin/plugin.json` na raiz, `source: "./"`), contendo uma skill (`skills/new-project/SKILL.md`) que faz o pré-voo, pergunta os dados do projeto e orquestra `gh`/`git` para materializar o repositório novo a partir do template.

**Tech Stack:** GitHub CLI (`gh`), Bash, Node.js (para validação JSON e o `overview.sh`/`setup-board.sh`, que já são Bash), Claude Code Plugin format (`.claude-plugin/*.json`, `skills/*/SKILL.md`).

## Global Constraints

- Ambos os repositórios são **públicos** — decidido no design; nenhum arquivo deve conter segredo real.
- Dependências de plugin (`superpowers`, `claude-mem`) são **checadas, nunca vendorizadas** — o pré-voo só verifica presença e oferece instalar a partir das marketplaces oficiais (`claude-plugins-official`, `thedotmack`).
- Credenciais de deploy real (SSH, tokens Vercel) **nunca são geradas/injetadas automaticamente** — a skill documenta o próximo passo manual.
- `sogest-project-template` não deve conter nenhum arquivo de metadado de plugin (`.claude-plugin/`, `skills/`) — separação estrita entre os dois repositórios.
- Todo arquivo Bash novo passa por `bash -n <arquivo>` (checagem de sintaxe) antes do commit.
- Todo arquivo JSON novo passa por `node -e "JSON.parse(require('fs').readFileSync('<arquivo>','utf-8'))"` antes do commit.

---

## File Structure

```
sogest-ai-project-method/                      (repo já existe, local + GitHub)
├── .claude-plugin/
│   ├── marketplace.json                        NOVO
│   └── plugin.json                              NOVO
├── skills/
│   └── new-project/
│       ├── SKILL.md                              NOVO
│       └── references/
│           ├── deploy-ssh.md                     NOVO
│           ├── deploy-vercel.md                  NOVO
│           └── deploy-manual.md                  NOVO
└── README.md                                     NOVO

sogest-project-template/                        (repo NOVO, público, GitHub Template Repository)
├── README.md
├── AGENTS.md
├── CONTRIBUTING.md
├── docs/
│   ├── PROJECT.md
│   ├── glossary.md
│   ├── WORKLIST.md
│   ├── how-we-build-a-module.md
│   └── adr/README.md
├── variants/
│   ├── deploy-ssh/{DEPLOYMENT.md, workflows/deploy-dev.yml, workflows/deploy-test.yml, workflows/deploy-prod.yml}
│   ├── deploy-vercel/{DEPLOYMENT.md, workflows/deploy.yml}
│   └── deploy-manual/{DEPLOYMENT.md}
├── .github/
│   ├── ISSUE_TEMPLATE/{bug_report.yml, suggestion.yml, config.yml}
│   └── workflows/ci.yml
└── scripts/
    ├── overview.sh
    └── setup-board.sh
```

`variants/` é uma pequena correção sobre o design original: o `DEPLOYMENT.md` também muda de conteúdo por modelo de deploy (não só os workflows), então cada modelo ganha uma pasta própria com doc + workflow(s) juntos. A skill copia a pasta escolhida para `docs/DEPLOYMENT.md` + `.github/workflows/`, depois apaga `variants/` inteiro — nunca sobra no repositório final.

---

### Task 1: Manifesto do plugin (`sogest-ai-project-method`)

**Files:**
- Create: `.claude-plugin/marketplace.json`
- Create: `.claude-plugin/plugin.json`
- Create: `README.md`

**Interfaces:**
- Produces: identidade do plugin (`name: "sogest-ai-project-method"`) que a Task 9 vai referenciar na skill.

- [ ] **Step 1: Criar `.claude-plugin/marketplace.json`**

```json
{
  "name": "sogest-ai-project-method",
  "description": "Marketplace do instalador de gestão de projetos com IA + GitHub da Sogest",
  "owner": {
    "name": "Pedro Mondardo Sartori",
    "email": "psartori@sogest.com.br"
  },
  "plugins": [
    {
      "name": "sogest-ai-project-method",
      "description": "Instala a estrutura de gestão de projetos da Sogest (labels, board, ADRs, método) num repositório GitHub novo, a partir de um onboarding interativo",
      "version": "0.1.0",
      "source": "./",
      "author": {
        "name": "Pedro Mondardo Sartori",
        "email": "psartori@sogest.com.br"
      }
    }
  ]
}
```

- [ ] **Step 2: Criar `.claude-plugin/plugin.json`**

```json
{
  "name": "sogest-ai-project-method",
  "description": "Instala a estrutura de gestão de projetos da Sogest (labels, board, ADRs, método) num repositório GitHub novo, a partir de um onboarding interativo",
  "version": "0.1.0",
  "author": {
    "name": "Pedro Mondardo Sartori",
    "email": "psartori@sogest.com.br"
  },
  "homepage": "https://github.com/pmsartori/sogest-ai-project-method",
  "repository": "https://github.com/pmsartori/sogest-ai-project-method",
  "license": "MIT",
  "keywords": ["project-management", "github", "onboarding", "sogest"]
}
```

- [ ] **Step 3: Criar `README.md`**

```markdown
# Sogest AI Project Method

Instalador de gestão de projetos com IA + GitHub. Roda a skill `new-project`
para criar um repositório novo já organizado: labels, board, milestones,
ADRs, e o método brainstorm→spec→plan→code→review.

## Instalar

```
/plugin marketplace add pmsartori/sogest-ai-project-method
/plugin install sogest-ai-project-method
```

## Usar

Dentro de uma pasta vazia, invoque a skill `new-project`. Ela faz o
pré-voo (checa `gh`, autenticação, e os plugins `superpowers`/`claude-mem`),
pergunta os dados do projeto, e cria o repositório a partir de
[`sogest-project-template`](https://github.com/pmsartori/sogest-project-template).

Documentação de design: `docs/superpowers/specs/`.
```

- [ ] **Step 4: Validar os JSONs**

Run: `node -e "JSON.parse(require('fs').readFileSync('.claude-plugin/marketplace.json','utf-8')); JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json','utf-8')); console.log('ok')"`
Expected: `ok`

- [ ] **Step 5: Commit**

```bash
git add .claude-plugin README.md
git commit -m "feat: add plugin manifest for sogest-ai-project-method"
```

---

### Task 2: Criar e marcar `sogest-project-template` como GitHub Template Repository

**Files:**
- Create (novo repo local): `sogest-project-template/README.md`

**Interfaces:**
- Produces: repositório `pmsartori/sogest-project-template` existente no GitHub, com `is_template: true` — Tasks 3–8 commitam dentro dele.

- [ ] **Step 1: Criar a pasta local e o README mínimo**

```bash
mkdir -p "/c/Users/PedroSartori-Sogest/sogest-project-template"
cd "/c/Users/PedroSartori-Sogest/sogest-project-template"
git init -b main
```

`-b main` é obrigatório aqui — sem ele, o branch inicial herda o padrão da
config global do git (que nesta máquina é `master`, não `main`), e todo o
resto do template (AGENTS.md, CONTRIBUTING.md, os workflows de deploy)
assume `main` como branch de produção.

Conteúdo de `README.md`:

```markdown
# [PROJETO]

[DESCRICAO_UMA_LINHA]

Este repositório foi criado a partir do template
[`sogest-project-template`](https://github.com/pmsartori/sogest-project-template)
pelo instalador `sogest-ai-project-method`. Leia `AGENTS.md` e
`docs/PROJECT.md` antes de contribuir.
```

- [ ] **Step 2: Commit inicial**

```bash
git add README.md
git commit -m "chore: scaffold sogest-project-template"
```

- [ ] **Step 3: Criar o repositório no GitHub e marcar como template**

```bash
gh repo create pmsartori/sogest-project-template --public --source=. --remote=origin --description "Template GitHub para projetos de software da Sogest AI Project Method" --push
gh api -X PATCH repos/pmsartori/sogest-project-template -f is_template=true
```

- [ ] **Step 4: Verificar que a flag de template está ativa**

Run: `gh api repos/pmsartori/sogest-project-template --jq .is_template`
Expected: `true`

---

### Task 3: Docs genéricos (`AGENTS.md`, `CONTRIBUTING.md`, `docs/PROJECT.md`, `docs/glossary.md`, `docs/WORKLIST.md`, `docs/how-we-build-a-module.md`, `docs/adr/README.md`)

**Files:**
- Create: `sogest-project-template/AGENTS.md`
- Create: `sogest-project-template/CONTRIBUTING.md`
- Create: `sogest-project-template/docs/PROJECT.md`
- Create: `sogest-project-template/docs/glossary.md`
- Create: `sogest-project-template/docs/WORKLIST.md`
- Create: `sogest-project-template/docs/how-we-build-a-module.md`
- Create: `sogest-project-template/docs/adr/README.md`
- Create: `sogest-project-template/docs/superpowers/specs/.gitkeep`
- Create: `sogest-project-template/docs/superpowers/plans/.gitkeep`

**Interfaces:**
- Produces: os placeholders `[PROJETO]`, `[DESCRICAO_UMA_LINHA]`, `[MODULO_1..N]`, `[SPRINT_DIAS]` que a Task 9 (skill) substitui na Fase 2.

- [ ] **Step 1: Criar `AGENTS.md`**

```markdown
# [PROJETO] — regras do projeto (leia primeiro)

[DESCRICAO_UMA_LINHA]

Este arquivo é a memória de projeto que todo desenvolvedor e assistente de
IA deve ler antes de trabalhar aqui. Docs completos: [`README.md`](README.md),
[`CONTRIBUTING.md`](CONTRIBUTING.md).

## Rastreamento do projeto

Trabalho é organizado em **módulos** (`area:` labels) → **epics** (issues +
sub-issues) → **sprints** (milestones), num **board**
(Triage→Backlog→Ready→In Progress→In Review→Done). Guia completo:
[`docs/PROJECT.md`](docs/PROJECT.md).

- **Visão geral do projeto:** rode [`scripts/overview.sh`](scripts/overview.sh)
  (sprint atual, progresso dos epics, trabalho por módulo, inbox de triagem).
- **Minhas tarefas:** `gh issue list --assignee @me`. **Um módulo:**
  `gh issue list --label area:<modulo>`. **Esta sprint:**
  `gh issue list --milestone "Sprint N"`.
- **Pegar um ticket:** self-assign + label `status:in-progress` + PR
  rascunho (`Closes #n`) antes de codar. **Sugerir:** issue nova →
  template 💡 Sugestão (cai em Triage).

## Módulos deste projeto

[LISTA_MODULOS_COM_EPICS]

## Regras de ouro

1. **Edite `develop` apenas.** Branch a partir dele, PR de volta pra ele.
   `test` e `main` são alvos de promoção — nunca push direto.
2. **Produção é liberada explicitamente por um humano.** Nunca faça deploy
   automático pra prod nem por iniciativa própria.
3. **TypeScript/linguagem principal em modo estrito.** Arquivos focados.
4. Consulte [`docs/adr/`](docs/adr/README.md) antes de mudar um padrão já
   assentado — e adicione um ADR quando tomar uma decisão que um engenheiro
   futuro possa questionar ou desfazer.

## Ambientes

[TABELA_AMBIENTES]
```

- [ ] **Step 2: Criar `CONTRIBUTING.md`**

```markdown
# Contribuindo & Fluxo de trabalho

Como trabalhamos no [PROJETO].

## Acesso

- **Código:** GitHub `pmsartori/[SLUG_PROJETO]` (privado ou público, ver
  README). Peça convite de colaborador ao dono do repositório.
- **Segredos:** nunca commitar credenciais reais. `.env*` é gitignored.

## Branching & pull requests

Trunk-based, branches curtas:

```
main            sempre deployável; protegida (PR + CI verde + 1 review)
develop         você trabalha aqui
feature/<nome>  sua branch de trabalho, a partir de develop
```

1. `git switch -c feature/descricao-curta`
2. Commits pequenos e focados. Escreva teste para todo comportamento
   novo/alterado.
3. Abra PR contra `develop`. CI precisa passar e 1 revisor aprovar.
4. Squash-merge pra `develop`. **Você só edita `develop`** — `test` e
   `main` são alvos de promoção, nunca push direto.

## Rastreamento de trabalho

- Trabalho é rastreado como **GitHub Issues**, organizado no **Project
  board** (Triage → Backlog → Ready → In Progress → In Review → Done).
- Pegue uma issue, self-assign, mova pra *In Progress*, referencie no PR
  (`Closes #NN`).

### Reivindicando trabalho — pra não pisar no pé de ninguém

1. **Self-assign** — `gh issue edit <n> --add-assignee @me`.
2. **Label `status:in-progress`** — `gh issue edit <n> --add-label status:in-progress`.
3. **Abra um PR _rascunho_ imediatamente**, antes do código existir —
   `gh pr create --draft --fill --base develop`.

**Ver quem trabalha em quê:** `gh issue list --label status:in-progress`
ou `gh issue list --assignee @me`.
```

- [ ] **Step 3: Criar `docs/PROJECT.md`** (o manual — deliverable 1)

```markdown
# Como rodamos o projeto (módulos · epics · sprints · board)

Guia único de como o trabalho é organizado e rastreado em [PROJETO]. Humanos
e assistentes de IA devem ler isto para responder "qual o estado do
projeto?" e "como eu pego / sugiro / rastreio trabalho?".

## Ver a visão geral — duas formas

- **Peça ao seu assistente:** *"me dá a visão geral do projeto"* → ele roda
  [`scripts/overview.sh`](../scripts/overview.sh) e imprime a sprint atual,
  progresso dos epics, trabalho por módulo, e o inbox de triagem.
- **Web:**
  - **Board (visual):** https://github.com/users/pmsartori/projects/N
  - **Sprints (milestones):** https://github.com/pmsartori/[SLUG_PROJETO]/milestones
  - **Todas as issues:** https://github.com/pmsartori/[SLUG_PROJETO]/issues

## A estrutura

Quatro níveis, do mais amplo ao mais granular: **Módulo → Epic → Task →
Sub-task**.

```
Módulo (label area:)
 └─ Epic          issue, label `epic`, um por módulo
     └─ Task      sub-issue do epic — unidade que alguém pega para trabalhar
         └─ Sub-task   sub-issue da task — só quando uma task precisa ser dividida
```

Módulos deste projeto e seu epic — **um epic por módulo, sempre**:

[TABELA_MODULOS_EPICS]

| Conceito | Primitivo do GitHub | Como usar |
|---|---|---|
| **Módulo** | label `area:` | Toda issue recebe uma — ver tabela acima. |
| **Epic** | issue com label `epic` + **sub-issues** | Filhos formam a barra de progresso. Antes de criar um epic novo, cheque esta tabela primeiro. |
| **Task** | **sub-issue de um epic** | A unidade normal de trabalho. |
| **Sub-task** | **sub-issue de uma task** | Só quando a task precisa de checklist próprio. |
| **Sprint** | **Milestone** com data | Ex.: "Sprint 1". |
| **Status/dono** | **Project board** + **assignee** | Triage → Backlog → Ready → In Progress → In Review → Done. |

## Comandos do dia a dia

```bash
scripts/overview.sh                       # visão geral completa
gh issue list --milestone "Sprint 1"       # esta sprint + donos
gh issue list --assignee @me               # minhas tarefas
gh issue list --label area:<modulo>        # um módulo
gh issue list --label needs-triage         # inbox de sugestões
gh issue list --label status:in-progress   # o que está sendo feito agora
```

## Os fluxos

**Pegar trabalho:**
1. `gh issue edit <n> --add-assignee @me --add-label status:in-progress`
2. Branch `feat/<n>-slug`, abra PR **rascunho imediatamente** (`Closes #<n>`).
3. Mova o card pra **In Progress**.

**Sugerir uma feature:** issue nova → *💡 Sugestão de feature* (ou
`gh issue create --label needs-triage,suggestion`). Cai em **Triage**.

**Nova task ou sub-task:**
```bash
gh issue create --title "…" --label area:<modulo>
gh api repos/pmsartori/[SLUG_PROJETO]/issues/<pai>/sub_issues -F sub_issue_id=$(gh api repos/pmsartori/[SLUG_PROJETO]/issues/<nova> --jq '.id')
```

**Novo epic** — só se um módulo genuinamente ainda não tem um:
```bash
gh issue create --title "[EPIC] …" --label epic,area:<modulo>
```
```

- [ ] **Step 4: Criar `docs/glossary.md`**

```markdown
# Glossário

Vocabulário do domínio deste projeto — preencha conforme os termos surgem,
pra que humanos e IA usem as mesmas palavras com o mesmo significado.

| Termo | Significado |
|---|---|
| _(preencher)_ | _(preencher)_ |
```

- [ ] **Step 5: Criar `docs/WORKLIST.md`**

```markdown
# Roadmap

Lista de features, fases, e o fluxo de pegar/sugerir ticket. Ver
[`docs/PROJECT.md`](PROJECT.md) para o guia completo de como o trabalho é
rastreado.

## Fases

- [ ] _(preencher a primeira fase/marco do projeto)_
```

- [ ] **Step 6: Criar `docs/how-we-build-a-module.md`**

```markdown
# Como construímos (o método)

O loop usado para todo trabalho não-trivial neste projeto.

```
  ideia ─▶ brainstorm ─▶ spec ─▶ plan ─▶ code+test ─▶ review/refine ─▶ merge
          (design)     (doc)   (tasks)  (subagents)   (olhos frescos)
```

| Estágio | Skill | Saída | Gate antes de avançar |
|---|---|---|---|
| **1. Brainstorm** | `superpowers:brainstorming` | Entendimento compartilhado + 2-3 abordagens com recomendação | Design apresentado e **aprovado pelo humano** |
| **2. Spec** | (brainstorming escreve) | `docs/superpowers/specs/YYYY-MM-DD-<topico>-design.md`, commitado | Humano revisou a spec escrita |
| **3. Plan** | `superpowers:writing-plans` | `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` — tasks TDD pequenas | Auto-revisão do plano passa |
| **4. Code + test** | `superpowers:subagent-driven-development` | Commits, cada task test-first, revisada entre tasks | Todas as tasks verdes + reviews limpas |
| **5. Refine** | review da branch inteira | Correções do que a review encontrou | Review recomenda merge |

## Por que essa ordem

- Design antes de código evita o retrabalho mais caro (construir a coisa
  errada).
- Spec + plan escritos dão contexto completo a um agente ou dev novo, sem
  deriva — o mesmo motivo de manter [ADRs](adr/README.md) e o
  [glossário](glossary.md).
```

- [ ] **Step 7: Criar `docs/adr/README.md`**

```markdown
# Architecture Decision Records (ADRs)

Um arquivo curto por decisão significativa, capturando **o porquê** — para
que trabalho futuro (humano ou IA) não relitigue escolhas já assentadas nem
desfaça sem querer o motivo de algo ter sido feito de certo jeito.

**Quando criar um:** você tomou uma decisão que um engenheiro razoável
poderia questionar ou desfazer depois — uma tecnologia, um padrão, uma
restrição, um trade-off. Se "por que é feito assim?" tem resposta
não-óbvia, escreva.

**Formato** (caiba em uma página): `NNNN-titulo-curto.md`
```
# NNNN. Título

**Status:** accepted | superseded by NNNN | deprecated
**Date:** YYYY-MM-DD

## Contexto
O que forçou a decisão — restrições, o problema, o que sabíamos.

## Decisão
O que escolhemos, dito diretamente.

## Consequências
O que isso facilita, o que dificulta, no que prestar atenção.
```

Suceder em vez de editar: uma decisão que muda ganha um ADR **novo**
marcando o antigo como superseded — o histórico é o ponto.

## Índice

_(preencher conforme ADRs forem criados)_
```

- [ ] **Step 8: Criar os `.gitkeep` de specs/plans**

```bash
mkdir -p docs/superpowers/specs docs/superpowers/plans
touch docs/superpowers/specs/.gitkeep docs/superpowers/plans/.gitkeep
```

- [ ] **Step 9: Verificar que os placeholders existem (proxy de teste)**

Run: `grep -rl '\[PROJETO\]' AGENTS.md CONTRIBUTING.md README.md docs/PROJECT.md`
Expected: lista os 4 arquivos (confirma que a Task 9/skill tem o que substituir)

- [ ] **Step 10: Commit**

```bash
git add AGENTS.md CONTRIBUTING.md docs
git commit -m "docs: add generic project docs skeleton to template"
git push
```

---

### Task 4: Variantes de deploy (`variants/deploy-ssh`, `variants/deploy-vercel`, `variants/deploy-manual`)

**Files:**
- Create: `sogest-project-template/variants/deploy-ssh/DEPLOYMENT.md`
- Create: `sogest-project-template/variants/deploy-ssh/workflows/deploy-dev.yml`
- Create: `sogest-project-template/variants/deploy-ssh/workflows/deploy-test.yml`
- Create: `sogest-project-template/variants/deploy-ssh/workflows/deploy-prod.yml`
- Create: `sogest-project-template/variants/deploy-vercel/DEPLOYMENT.md`
- Create: `sogest-project-template/variants/deploy-vercel/workflows/deploy.yml`
- Create: `sogest-project-template/variants/deploy-manual/DEPLOYMENT.md`

**Interfaces:**
- Produces: as 3 pastas que a Task 9 (Fase 2, passo "escolher modelo de deploy") copia para `docs/DEPLOYMENT.md` e `.github/workflows/`.

- [ ] **Step 1: `variants/deploy-ssh/DEPLOYMENT.md`**

```markdown
# Deploy — SSH em servidor próprio

Pipeline de 3 ambientes, um host, um cluster Postgres (ou banco
equivalente), três bases:

```
feature/* ──PR──▶ develop ──promoção──▶ test ──gate (humano)──▶ main
                    DEV               TEST                     PROD
             deploy auto no push  deploy auto no push    disparo manual apenas
```

- **Edite `develop` apenas.** `test`/`main` são alvos de promoção.
- `develop → test`: fast-forward automático (workflow
  `deploy-test.yml`).
- `test → main`: exige PR revisado. Deploy de prod é **sempre disparo
  manual humano** (`gh workflow run "Deploy · prod" --ref main`).
- Configure os secrets do repositório antes do primeiro deploy:
  `SSH_HOST`, `SSH_USER`, `SSH_KEY`, `DEPLOY_PATH` — **este passo é
  manual**, o instalador não gera nem injeta credenciais.

| Env | Branch | Deploy |
|---|---|---|
| dev | `develop` | automático no push |
| test | `test` | automático na promoção |
| **prod** | `main` | **manual apenas** |
```

- [ ] **Step 2: `variants/deploy-ssh/workflows/deploy-dev.yml`**

```yaml
name: Deploy · dev
on:
  push:
    branches: [develop]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd ${{ secrets.DEPLOY_PATH }}/dev
            git pull origin develop
            # ajuste os comandos de build/restart deste projeto aqui
```

- [ ] **Step 3: `variants/deploy-ssh/workflows/deploy-test.yml`**

```yaml
name: Promote develop → test
on:
  push:
    branches: [develop]
jobs:
  promote:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Fast-forward test
        run: |
          git fetch origin develop:develop
          git checkout test
          git merge --ff-only develop
          git push origin test
```

- [ ] **Step 4: `variants/deploy-ssh/workflows/deploy-prod.yml`**

```yaml
name: Deploy · prod
on:
  workflow_dispatch: {}
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
        with:
          ref: main
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd ${{ secrets.DEPLOY_PATH }}/prod
            git pull origin main
            # ajuste os comandos de build/restart deste projeto aqui
```

- [ ] **Step 5: `variants/deploy-vercel/DEPLOYMENT.md`**

```markdown
# Deploy — Vercel / Netlify

A plataforma faz o deploy automaticamente a cada push, conectada
diretamente ao repositório GitHub via o app oficial dela — não há
workflow de deploy próprio a manter.

## Configuração (manual, uma vez)

1. Conecte o repositório em vercel.com (ou netlify.com) → "Import Project".
2. Configure a branch de produção como `main`.
3. Configure `develop` e `test` como branches de preview, se aplicável.
4. Variáveis de ambiente/segredos são configuradas no painel da
   plataforma — nunca commitadas no repositório.

## Fluxo de branch (igual aos outros modelos)

`feature/* → PR → develop → promoção → test → PR revisado → main`. A
plataforma gera uma preview URL automática pra cada PR.
```

- [ ] **Step 6: `variants/deploy-vercel/workflows/deploy.yml`**

```yaml
name: CI (Vercel cuida do deploy)
on:
  pull_request: {}
jobs:
  note:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy é feito pelo app oficial da Vercel/Netlify conectado a este repositório — nenhuma action de deploy é necessária aqui."
```

- [ ] **Step 7: `variants/deploy-manual/DEPLOYMENT.md`**

```markdown
# Deploy — manual

Este projeto ainda não tem pipeline de deploy automatizado. Documente
aqui, à mão, como o deploy é feito hoje (host, comando, quem faz) assim
que isso for definido.

## Fluxo de branch (mesmo dos outros modelos)

`feature/* → PR → develop → promoção → test → PR revisado → main`. A
promoção entre ambientes e o deploy em si são manuais até um modelo ser
escolhido.
```

- [ ] **Step 8: Verificar YAML válido**

Run: `node -e "require('fs').readdirSync('variants',{recursive:true}).filter(f=>f.endsWith('.yml')).forEach(f=>console.log(f))"`
Expected: lista os 4 arquivos `.yml` criados (checagem de que existem; parsing YAML completo não é necessário pois GitHub Actions valida no push)

- [ ] **Step 9: Commit**

```bash
git add variants
git commit -m "feat: add deploy model variants (ssh, vercel, manual)"
git push
```

---

### Task 5: Issue templates e CI genérico

**Files:**
- Create: `sogest-project-template/.github/ISSUE_TEMPLATE/bug_report.yml`
- Create: `sogest-project-template/.github/ISSUE_TEMPLATE/suggestion.yml`
- Create: `sogest-project-template/.github/ISSUE_TEMPLATE/config.yml`
- Create: `sogest-project-template/.github/workflows/ci.yml`

**Interfaces:**
- Consumes: nenhuma (arquivos estáticos, mesmos independente do modelo de deploy).

- [ ] **Step 1: `bug_report.yml`**

```yaml
name: 🐛 Bug report
description: Reportar um comportamento incorreto
labels: ["bug", "needs-triage"]
body:
  - type: textarea
    id: what-happened
    attributes:
      label: O que aconteceu?
    validations:
      required: true
  - type: textarea
    id: expected
    attributes:
      label: O que era esperado?
    validations:
      required: true
```

- [ ] **Step 2: `suggestion.yml`**

```yaml
name: 💡 Sugestão de feature
description: Propor uma ideia nova — cai em Triage
labels: ["needs-triage", "suggestion"]
body:
  - type: textarea
    id: problem
    attributes:
      label: Que problema isso resolve?
    validations:
      required: true
  - type: textarea
    id: proposal
    attributes:
      label: Proposta
    validations:
      required: false
```

- [ ] **Step 3: `config.yml`**

```yaml
blank_issues_enabled: true
contact_links:
  - name: 📋 Roadmap & como trabalhamos
    url: https://github.com/pmsartori/[SLUG_PROJETO]/blob/develop/docs/WORKLIST.md
    about: A lista de features, fases, e o fluxo de pegar/sugerir ticket.
```

- [ ] **Step 4: `.github/workflows/ci.yml`**

```yaml
name: CI
on:
  pull_request: {}
  push:
    branches: [develop]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Substitua por lint/test/build reais deste projeto (ex.: npm ci && npm test && npm run build)"
```

- [ ] **Step 5: Commit**

```bash
git add .github
git commit -m "feat: add issue templates and generic CI workflow"
git push
```

---

### Task 6: `scripts/overview.sh` e `scripts/setup-board.sh`

**Files:**
- Create: `sogest-project-template/scripts/overview.sh`
- Create: `sogest-project-template/scripts/setup-board.sh`

**Interfaces:**
- Consumes: `gh` CLI autenticado, labels/milestones já existentes no repo (criadas pela Task 9 antes de rodar `overview.sh`).
- Produces: `setup-board.sh` é reaproveitável tanto pela Task 9 (skill) quanto manualmente.

- [ ] **Step 1: `scripts/overview.sh`**

```bash
#!/usr/bin/env bash
# Visão geral do projeto: sprint atual, progresso por epic, trabalho por
# módulo, inbox de triagem. Uso: scripts/overview.sh ["Sprint N"]
set -euo pipefail

SPRINT="${1:-}"

echo "== Sprint =="
if [ -n "$SPRINT" ]; then
  gh issue list --milestone "$SPRINT" --json number,title,assignees,state \
    --template '{{range .}}#{{.number}} {{.title}} ({{range .assignees}}{{.login}} {{end}}){{"\n"}}{{end}}'
else
  gh api graphql -f query='query{ repository(owner:"pmsartori",name:"[SLUG_PROJETO]"){ milestones(first:1, states:OPEN, orderBy:{field:DUE_DATE,direction:ASC}){ nodes{ title dueOn } } } }' \
    --jq '.data.repository.milestones.nodes[0].title // "sem sprint aberta"'
fi

echo
echo "== Epics (progresso) =="
gh issue list --label epic --state all --json number,title,state \
  --template '{{range .}}#{{.number}} {{.title}} [{{.state}}]{{"\n"}}{{end}}'

echo
echo "== Trabalho por módulo =="
for label in $(gh label list --json name --jq '.[].name' | grep '^area:'); do
  count=$(gh issue list --label "$label" --state open --json number --jq 'length')
  echo "$label: $count aberta(s)"
done

echo
echo "== Inbox de triagem =="
gh issue list --label needs-triage --json number,title \
  --template '{{range .}}#{{.number}} {{.title}}{{"\n"}}{{end}}'
```

- [ ] **Step 2: `scripts/setup-board.sh`** (parametrizado)

```bash
#!/usr/bin/env bash
# Cria o Project board do projeto e carrega toda issue aberta nele.
#
# PRE-REQUISITO (uma vez, dono da conta):
#   gh auth refresh -s project,read:project
#
# Uso:  OWNER=pmsartori REPO=pmsartori/[SLUG_PROJETO] TITLE="[PROJETO] Roadmap" scripts/setup-board.sh
set -euo pipefail

: "${OWNER:?defina OWNER}"
: "${REPO:?defina REPO}"
: "${TITLE:?defina TITLE}"

echo "==> criando board '$TITLE'"
NUM=$(gh project create --owner "$OWNER" --title "$TITLE" --format json | jq -r '.number')
echo "    board #$NUM"

STATUS_FIELD=$(gh project field-list "$NUM" --owner "$OWNER" --format json \
  | jq -r '.fields[] | select(.name=="Status") | .id')

echo "==> configurando colunas do Status"
gh api graphql -f query='
  mutation($field:ID!) {
    updateProjectV2Field(input:{
      fieldId:$field,
      singleSelectOptions:[
        {name:"Triage",      color:YELLOW, description:"Aguardando aceite/recusa"},
        {name:"Backlog",     color:GRAY,   description:"Aceito, não iniciado"},
        {name:"Ready",       color:BLUE,   description:"Definido, pronto pra pegar"},
        {name:"In Progress", color:PURPLE, description:"Alguém está fazendo"},
        {name:"In Review",   color:ORANGE, description:"PR aberto"},
        {name:"Done",        color:GREEN,  description:"Mergeado / entregue"}
      ]
    }){ projectV2Field { ... on ProjectV2SingleSelectField { id } } }
  }' -f field="$STATUS_FIELD" >/dev/null
echo "    colunas: Triage / Backlog / Ready / In Progress / In Review / Done"

echo "==> adicionando issues abertas"
gh issue list --repo "$REPO" --state open --limit 200 --json url -q '.[].url' \
  | while read -r url; do gh project item-add "$NUM" --owner "$OWNER" --url "$url" >/dev/null && echo "    + $url"; done

echo
echo "Pronto. Board: https://github.com/users/$OWNER/projects/$NUM"
```

- [ ] **Step 3: Checar sintaxe Bash**

Run: `bash -n scripts/overview.sh && bash -n scripts/setup-board.sh && echo ok`
Expected: `ok`

- [ ] **Step 4: Dar permissão de execução e commitar**

```bash
chmod +x scripts/overview.sh scripts/setup-board.sh
git add scripts
git commit -m "feat: add overview.sh and setup-board.sh scripts"
git push
```

---

### Task 7: Skill `new-project` — pré-voo

**Files:**
- Create: `sogest-ai-project-method/skills/new-project/SKILL.md`

**Interfaces:**
- Consumes: nada (primeiro conteúdo do arquivo).
- Produces: seção "Pré-voo" que a Task 8 (Fase 1/2) vai continuar no mesmo arquivo.

- [ ] **Step 1: Criar `skills/new-project/SKILL.md` com frontmatter + pré-voo**

```markdown
---
name: new-project
description: "Cria um projeto novo no GitHub já organizado com labels, board, milestones, ADRs e o método brainstorm-spec-plan-code da Sogest. Use quando o usuário quiser começar um projeto novo, iniciar um repositório, ou 'montar a estrutura' de algo novo."
---

# new-project — instalador Sogest AI Project Method

Cria um repositório GitHub novo a partir de `pmsartori/sogest-project-template`
(tipo Software) ou `pmsartori/sogest-methodology-template` (tipo Metodologia
e Gestão — ver `references/`), já com labels, board, milestones, epics e o
método de trabalho configurados.

## Fase 0 — Pré-voo

Rode estas checagens, nesta ordem, ANTES de fazer qualquer pergunta sobre o
projeto. Pare e instrua o usuário em qualquer falha — não prossiga sem
resolver.

1. **`gh` instalado?**
   Run: `gh --version`
   Se falhar: instrua a instalar via https://cli.github.com e pare.

2. **`gh` autenticado?**
   Run: `gh auth status`
   Se não autenticado: instrua `gh auth login` e pare — login é
   interativo, não pode ser feito por você.

3. **Escopo `project` presente?**
   Run: `gh auth status` (leia a linha "Token scopes")
   Se faltar `project`: instrua `gh auth refresh -s project,read:project`
   e pare.

4. **Plugin `superpowers` instalado?**
   Verifique se `superpowers@claude-plugins-official` aparece na lista de
   plugins instalados do usuário. Se não estiver, ofereça rodar:
   `/plugin marketplace add claude-plugins-official` seguido de
   `/plugin install superpowers`. **Nunca copie os arquivos do
   superpowers** — só instale via marketplace oficial.

5. **Plugin `claude-mem` instalado?**
   Mesma checagem, para `claude-mem@thedotmack`. Se ausente, ofereça
   `/plugin marketplace add thedotmack` + `/plugin install claude-mem`.

Só avance para a Fase 1 depois que as 5 checagens passarem.
```

- [ ] **Step 2: Validar frontmatter YAML**

Run: `node -e "const fs=require('fs'); const c=fs.readFileSync('skills/new-project/SKILL.md','utf-8'); const m=c.match(/^---\n([\s\S]*?)\n---/); if(!m) throw new Error('sem frontmatter'); console.log('frontmatter ok:', m[1].includes('name: new-project'))"`
Expected: `frontmatter ok: true`

- [ ] **Step 3: Commit**

```bash
git add skills/new-project/SKILL.md
git commit -m "feat: add new-project skill with pre-flight checks"
```

---

### Task 8: Skill `new-project` — Fase 1 (perguntas) e Fase 2 (automação) para tipo Software

**Files:**
- Modify: `sogest-ai-project-method/skills/new-project/SKILL.md` (acrescentar seções)
- Create: `sogest-ai-project-method/skills/new-project/references/deploy-ssh.md`
- Create: `sogest-ai-project-method/skills/new-project/references/deploy-vercel.md`
- Create: `sogest-ai-project-method/skills/new-project/references/deploy-manual.md`

**Interfaces:**
- Consumes: pré-voo aprovado (Task 7); `pmsartori/sogest-project-template` publicado com `variants/` (Tasks 2–6).
- Produces: fluxo completo — este é o corpo principal que Fase 1 do tipo "Metodologia e Gestão" (plano futuro) vai estender com um branch novo na Step 1.

- [ ] **Step 1: Acrescentar Fase 1 ao `SKILL.md`**

```markdown

## Fase 1 — Perguntas

Pergunte uma de cada vez.

1. **Tipo de projeto:** Software ou Metodologia e Gestão? (Metodologia e
   Gestão está fora do escopo desta versão da skill — se escolhido, avise
   que ainda não está implementado e pare.)
2. **Nome do projeto** (vira o slug do repo, kebab-case) + descrição de
   uma linha.
3. **Privado ou público?** (recomendação padrão: público, para tipo
   Software.)
4. **Módulos do projeto** — lista livre, ex: "Tenancy, Reporting, Import".
   Cada um vira uma label `area:<slug>` + 1 epic.
5. **Modelo de deploy:** SSH próprio / Vercel-Netlify / Nenhum (manual).
   - Se SSH: pergunte host, usuário, caminho remoto. As credenciais reais
     (chave SSH) ficam como próximo passo manual — nunca peça a chave em
     texto no chat.
6. **Cadência de sprint** (ex.: 2 semanas) → define a data de vencimento
   do milestone "Sprint 1" (hoje + N dias).

Confirme um resumo das respostas antes de prosseguir para a Fase 2.
```

- [ ] **Step 2: Acrescentar Fase 2 ao `SKILL.md`**

```markdown

## Fase 2 — Automação

Execute nesta ordem exata. Pare e reporte se qualquer comando falhar.

1. **Criar o repositório a partir do template:**
   ```bash
   gh repo create pmsartori/<slug> --template pmsartori/sogest-project-template --public   # ou --private
   git clone https://github.com/pmsartori/<slug>.git
   cd <slug>
   ```

2. **Substituir os placeholders** em `README.md`, `AGENTS.md`,
   `CONTRIBUTING.md`, `docs/PROJECT.md`, `.github/ISSUE_TEMPLATE/config.yml`,
   `scripts/overview.sh`, `scripts/setup-board.sh`:
   - `[PROJETO]` → nome do projeto
   - `[DESCRICAO_UMA_LINHA]` → descrição
   - `[SLUG_PROJETO]` → slug do repo
   - `[LISTA_MODULOS_COM_EPICS]` e `[TABELA_MODULOS_EPICS]` → lista/tabela
     dos módulos informados (preencha o número do epic depois do Step 5)
   - `[TABELA_AMBIENTES]` → copie a tabela de ambientes do
     `variants/<modelo>/DEPLOYMENT.md` escolhido

3. **Aplicar o modelo de deploy escolhido** (consulte
   `references/deploy-<modelo>.md` para o passo a passo exato):
   ```bash
   cp variants/<modelo>/DEPLOYMENT.md docs/DEPLOYMENT.md
   mkdir -p .github/workflows
   cp variants/<modelo>/workflows/*.yml .github/workflows/ 2>/dev/null || true
   rm -rf variants
   ```

4. **Commit e push em `main`** (antes de criar `develop`/`test`, para que
   as duas nasçam já com o conteúdo preenchido, não com os placeholders
   crus do template):
   ```bash
   git add -A
   git commit -m "chore: fill onboarding placeholders"
   git push
   ```

5. **Criar `develop` e `test` a partir de `main`, e proteger `main`:**
   ```bash
   git checkout -b develop
   git push -u origin develop
   git checkout -b test
   git push -u origin test
   git checkout main
   gh api -X PUT repos/pmsartori/<slug>/branches/main/protection \
     -f required_status_checks='null' \
     -F enforce_admins=false \
     -f required_pull_request_reviews='{"required_approving_review_count":1}' \
     -f restrictions='null'
   git checkout develop
   ```
   Se o repositório for privado num plano GitHub Free, a API de branch
   protection pode retornar 403 (recurso de plano pago) — nesse caso,
   avise o usuário e trate como próximo passo manual em vez de falhar o
   fluxo inteiro. Termine este passo na branch `develop` — é onde o
   trabalho subsequente acontece.

6. **Criar as labels:**
   ```bash
   gh label create epic --color 5319e7 --description "Large multi-issue workstream"
   gh label create "status:ready" --color 5319E7
   gh label create "status:in-progress" --color 1D76DB
   gh label create "status:in-review" --color 0E8A16
   gh label create "status:blocked" --color B60205
   gh label create needs-triage --color FBCA04
   gh label create accepted --color 0E8A16
   gh label create suggestion --color C5DEF5
   gh label create "good first issue" --color 7057ff
   gh label create tech-debt --color d4c5f9
   gh label create security --color b60205
   # uma label area:<slug> por módulo informado na Fase 1
   for m in "${MODULOS[@]}"; do gh label create "area:$m" --color 1D76DB; done
   ```

7. **Criar 1 epic por módulo:**
   ```bash
   for m in "${MODULOS[@]}"; do
     gh issue create --title "[EPIC] $m" --label "epic,area:$m"
   done
   ```

8. **Criar o milestone da Sprint 1:**
   ```bash
   gh api repos/pmsartori/<slug>/milestones -f title="Sprint 1" -f due_on="<hoje+N_dias>T00:00:00Z"
   ```

9. **Criar o board:**
   ```bash
   OWNER=pmsartori REPO=pmsartori/<slug> TITLE="<Projeto> Roadmap" bash scripts/setup-board.sh
   ```

10. **Resumo final** — imprima para o usuário: link do repo, link do
    board, link dos milestones, e a lista de próximos passos manuais (ex.:
    "configure os secrets SSH_HOST/SSH_USER/SSH_KEY/DEPLOY_PATH antes do
    primeiro deploy" se o modelo escolhido foi SSH; branch protection
    manual se a API retornou 403).
```

- [ ] **Step 3: Criar `references/deploy-ssh.md`**

```markdown
# Modelo de deploy: SSH próprio

Depois de copiar `variants/deploy-ssh/*` (Fase 2, Step 3), configure no
repositório novo (Settings → Secrets and variables → Actions):

- `SSH_HOST` — IP ou domínio do servidor
- `SSH_USER` — usuário SSH
- `SSH_KEY` — chave privada (nunca peça isso em texto no chat; instrua o
  usuário a colar direto no campo do GitHub)
- `DEPLOY_PATH` — caminho base no servidor (os workflows assumem
  subpastas `dev/`, `test/`, `prod/` dentro dele)

Sem esses secrets configurados, `deploy-dev.yml` e `deploy-prod.yml`
falham no primeiro push — isso é esperado até a configuração manual ser
feita.
```

- [ ] **Step 4: Criar `references/deploy-vercel.md`**

```markdown
# Modelo de deploy: Vercel / Netlify

Depois de copiar `variants/deploy-vercel/*` (Fase 2, Step 3):

1. Acesse vercel.com (ou netlify.com), "Import Project", conecte
   `pmsartori/<slug>`.
2. Configure `main` como branch de produção.
3. Variáveis de ambiente são configuradas no painel da plataforma, nunca
   no repositório.

Não há secrets do GitHub Actions a configurar — o deploy roda pelo app
oficial da plataforma, fora do GitHub Actions.
```

- [ ] **Step 5: Criar `references/deploy-manual.md`**

```markdown
# Modelo de deploy: manual

Nenhuma configuração adicional é necessária agora. `docs/DEPLOYMENT.md`
já foi copiado como placeholder — atualize-o assim que o time decidir como
o deploy vai funcionar.
```

- [ ] **Step 6: Commit**

```bash
git add skills/new-project
git commit -m "feat: add Fase 1/2 onboarding flow and deploy references for tipo Software"
git push
```

---

### Task 9: Teste ponta a ponta (dry run real)

**Files:** nenhum arquivo novo — este task só executa e verifica.

**Interfaces:**
- Consumes: tudo das Tasks 1–8.

- [ ] **Step 1: Instalar o plugin localmente**

```
/plugin marketplace add pmsartori/sogest-ai-project-method
/plugin install sogest-ai-project-method
```

Expected: plugin aparece instalado (mesmo padrão de
`installed_plugins.json` verificado no início do projeto).

- [ ] **Step 2: Rodar a skill `new-project` numa pasta de teste, tipo Software, modelo de deploy Manual**

Crie o repositório de teste `pmsartori/sogest-method-e2e-test` respondendo
às perguntas da Fase 1 com dados de teste (módulo único "core", deploy
Manual, sprint de 7 dias).

- [ ] **Step 3: Verificar o resultado**

Run: `gh repo view pmsartori/sogest-method-e2e-test --json name,isPrivate`
Expected: repo existe, `isPrivate: false`

Run: `gh label list --repo pmsartori/sogest-method-e2e-test | grep -c "area:core\|epic\|status:"`
Expected: número > 0 (labels criadas)

Run: `gh issue list --repo pmsartori/sogest-method-e2e-test --label epic --json title`
Expected: 1 issue `[EPIC] core`

Run: `gh api repos/pmsartori/sogest-method-e2e-test/contents/docs/DEPLOYMENT.md --jq .name`
Expected: `DEPLOYMENT.md` (confirma que a variante manual foi copiada e
`variants/` não sobrou)

Run: `gh api repos/pmsartori/sogest-method-e2e-test/contents/variants 2>&1`
Expected: erro 404 (pasta `variants/` foi removida, como especificado)

- [ ] **Step 4: Limpar o repositório de teste**

```bash
gh repo delete pmsartori/sogest-method-e2e-test --yes
```

(Repositório efêmero criado só para este teste — apagar é seguro; não
apaga nada além do que este próprio task criou.)

- [ ] **Step 5: Reportar resultado ao usuário**

Resuma o que passou/falhou no dry run antes de considerar a Task 9
concluída.
