# Metodologia e Gestão (tipo de projeto) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implementar o segundo tipo de projeto do instalador — "Metodologia e Gestão" — cobrindo o repositório-template `sogest-methodology-template` e a extensão da skill `new-project` pra realmente ramificar por tipo (hoje ela só para e avisa "fora de escopo").

**Architecture:** Um segundo repositório GitHub Template Repository, público (estrutura genérica, sem dado de cliente), consumido pela mesma skill `new-project` já existente — que passa a ramificar em Fase 1 (perguntas condicionais por tipo) e Fase 2 (automação condicional por tipo), reaproveitando 100% do pré-voo, labels, epics, milestone e board já construídos.

**Tech Stack:** Markdown, Bash, Node.js + pacote `docx` (para `export-docx.js`, generalizando `gen-spec-docx.js` do Prisma).

## Global Constraints

- `sogest-methodology-template` é **público** (mesma razão do template de Software: estrutura genérica, sem segredo, evita convite manual) — só os PROJETOS GERADOS a partir dele nascem privados por padrão.
- Nenhum arquivo de metadado de plugin (`.claude-plugin/`, `skills/`) no template.
- Projetos gerados deste tipo **não têm pipeline de deploy** — sem `develop`/`test`, sem `.github/workflows` de deploy. Apenas `main`, protegida (PR + review).
- `export-docx.js` **nunca** é a fonte de verdade — sempre gera o `.docx` a partir do Markdown versionado, nunca o contrário.
- Todo arquivo Bash novo passa por `bash -n`. Todo `.js` novo tem pelo menos um teste real (não mockado) para sua lógica pura (classificação de linha Markdown), rodável com `node --test`.

---

## File Structure

```
sogest-methodology-template/                     (repo NOVO, público, GitHub Template Repository)
├── README.md
├── AGENTS.md
├── CONTRIBUTING.md
├── docs/
│   ├── PROJECT.md
│   ├── glossary.md
│   ├── how-we-conduct-a-deliverable.md
│   ├── decision-records/README.md
│   └── memoria-discursiva.md
├── templates/
│   ├── master-briefing-template.md
│   └── weekly-report-template.md
├── scripts/
│   ├── overview.sh                                (idêntico ao do template de Software)
│   ├── export-docx.js                             NOVO
│   └── export-docx.test.js                        NOVO
└── .github/
    └── ISSUE_TEMPLATE/{gap.yml, suggestion.yml, config.yml}

sogest-ai-project-method/
└── skills/new-project/SKILL.md                     MODIFICADO (Fase 1 + Fase 2 ramificam por tipo)
```

---

### Task 1: Criar e marcar `sogest-methodology-template` como GitHub Template Repository

**Files:**
- Create (novo repo local): `sogest-methodology-template/README.md`

**Interfaces:**
- Produces: repositório `pmsartori/sogest-methodology-template` no GitHub, `is_template: true` — Tasks 2–5 commitam dentro dele.

- [ ] **Step 1: Criar a pasta local, inicializar em `main`, e o README**

```bash
mkdir -p "/c/Users/PedroSartori-Sogest/sogest-methodology-template"
cd "/c/Users/PedroSartori-Sogest/sogest-methodology-template"
git init -b main
```

`-b main` é obrigatório — sem ele, o branch inicial herda `master` da config
global do git nesta máquina.

Conteúdo de `README.md`:

```markdown
# [PROJETO]

[DESCRICAO_UMA_LINHA]

Este repositório foi criado a partir do template
[`sogest-methodology-template`](https://github.com/pmsartori/sogest-methodology-template)
pelo instalador `sogest-ai-project-method` — tipo **Metodologia e Gestão**
(condução de projeto, sem software). Leia `AGENTS.md` e `docs/PROJECT.md`
antes de contribuir.
```

- [ ] **Step 2: Commit inicial**

```bash
git add README.md
git commit -m "chore: scaffold sogest-methodology-template"
```

- [ ] **Step 3: Criar o repositório no GitHub e marcar como template**

```bash
gh repo create pmsartori/sogest-methodology-template --public --source=. --remote=origin --description "Template GitHub para projetos de metodologia e gestão da Sogest AI Project Method" --push
gh api -X PATCH repos/pmsartori/sogest-methodology-template -f is_template=true
```

- [ ] **Step 4: Verificar**

Run: `gh api repos/pmsartori/sogest-methodology-template --jq '{is_template, default_branch}'`
Expected: `{"is_template":true,"default_branch":"main"}`

---

### Task 2: Docs do método (`docs/PROJECT.md`, `glossary.md`, `how-we-conduct-a-deliverable.md`, `decision-records/README.md`, `memoria-discursiva.md`)

**Files:**
- Create: `sogest-methodology-template/docs/PROJECT.md`
- Create: `sogest-methodology-template/docs/glossary.md`
- Create: `sogest-methodology-template/docs/how-we-conduct-a-deliverable.md`
- Create: `sogest-methodology-template/docs/decision-records/README.md`
- Create: `sogest-methodology-template/docs/memoria-discursiva.md`
- Create: `sogest-methodology-template/docs/superpowers/specs/.gitkeep`
- Create: `sogest-methodology-template/docs/superpowers/plans/.gitkeep`

**Interfaces:**
- Produces: os placeholders `[PROJETO]`, `[DESCRICAO_UMA_LINHA]`, `[SLUG_PROJETO]`, `[LISTA_MODULOS_COM_EPICS]`, `[TABELA_MODULOS_EPICS]` (mesmos tokens do template de Software — **sem** `[TABELA_AMBIENTES]`, pois este tipo não tem camada de deploy).

- [ ] **Step 1: Criar `docs/PROJECT.md`**

```markdown
# Como conduzimos o projeto (módulos · epics · sprints · board · gates)

Guia único de como o trabalho é organizado e rastreado em [PROJETO]. Humanos
e assistentes de IA devem ler isto para responder "qual o estado do
projeto?" e "como eu pego / sugiro / rastreio trabalho?".

## Ver a visão geral

- **Peça ao seu assistente:** *"me dá a visão geral do projeto"* → ele roda
  [`scripts/overview.sh`](../scripts/overview.sh).
- **Web:**
  - **Board (visual):** https://github.com/users/pmsartori/projects/N
  - **Sprints (milestones):** https://github.com/pmsartori/[SLUG_PROJETO]/milestones
  - **Todas as issues:** https://github.com/pmsartori/[SLUG_PROJETO]/issues

## A estrutura

Quatro níveis, do mais amplo ao mais granular: **Módulo → Epic → Task →
Sub-task** — mesma lógica de um projeto de software, só que aqui um
"módulo" é uma frente de trabalho de condução (ex.: Master Briefing,
Portfólio, Weekly Update) em vez de uma área de código.

Módulos deste projeto e seu epic — **um epic por módulo, sempre**:

[TABELA_MODULOS_EPICS]

| Conceito | Primitivo do GitHub | Como usar |
|---|---|---|
| **Módulo** | label `area:` | Toda issue recebe uma. |
| **Epic** | issue com label `epic` + sub-issues | Filhos formam a barra de progresso. |
| **Task** | sub-issue de um epic | A unidade normal de trabalho. |
| **Sub-task** | sub-issue de uma task | Só quando a task precisa de checklist próprio. |
| **Sprint** | Milestone com data | Ex.: "Sprint 1". |
| **Status/dono** | Project board + assignee | Triage → Backlog → Ready → In Progress → In Review → Done. |
| **Gate** | campo customizado do board | A etapa da jornada (ex.: Diagnóstico, Arquitetura, Produção) — ver `how-we-conduct-a-deliverable.md`. |

O Portfólio (visão executiva de todos os entregáveis, com Gate/Pilar/
Cluster/Impacto) não vira planilha separada — é o próprio board do GitHub:
crie *views* salvas agrupadas por Gate (visão executiva), filtradas
(resumo), e a lista completa (base total). O histórico de mudanças é o
log de atividade nativo de cada issue.

## Comandos do dia a dia

```bash
scripts/overview.sh
gh issue list --milestone "Sprint 1"
gh issue list --assignee @me
gh issue list --label area:<modulo>
gh issue list --label needs-triage
```

## Os fluxos

**Pegar trabalho:** self-assign + label `status:in-progress` + PR rascunho
(`Closes #<n>`), mesma disciplina de um projeto de software.

**Sugerir uma frente nova:** issue nova → template 💡 Sugestão (cai em
Triage).
```

- [ ] **Step 2: Criar `docs/glossary.md`**

```markdown
# Glossário

Vocabulário do domínio deste projeto — preencha conforme os termos surgem.

| Termo | Significado |
|---|---|
| _(preencher)_ | _(preencher)_ |
```

- [ ] **Step 3: Criar `docs/how-we-conduct-a-deliverable.md`**

```markdown
# Como conduzimos um entregável (o método)

A disciplina de gates que usávamos num OPS de chat solto, agora formalizada
em git — mesma ideia do `superpowers:brainstorming` pra software: **nunca
produzir o entregável final sem entendimento validado.**

| Etapa | Equivalente no método Git |
|---|---|
| Entrevista + Diagnóstico | `superpowers:brainstorming` — perguntas uma a uma, sem gerar entregável |
| Arquitetura | Apresentação do design em seções, aprovação |
| Gate antes de Produção | O gate que já existe no `brainstorming` — nada de doc final sem aprovação explícita |
| Produção | Spec + entregável final em Markdown, commitado |
| Captura Semanal (contínua; capturar ≠ atualizar) | Comentários acumulando numa Issue "semana N" |
| Consolidação Semanal ("agora pode rodar a atualização") | Fechar a issue da semana com um resumo — só quando pedido explicitamente |

## Onde cada entregável mora

- **Master Briefing** (contexto-mestre, governança) → `docs/PROJECT.md`
  preenchido, ou um documento dedicado em `docs/` se o projeto crescer.
- **Portfólio** → o Project board nativo (ver `docs/PROJECT.md`).
- **Memória Discursiva** (narrativa, relacional, sensível) →
  `docs/memoria-discursiva.md`.
- **Weekly Report** → preencha
  [`templates/weekly-report-template.md`](../templates/weekly-report-template.md)
  a cada consolidação.

## Exportar para Word (quando precisar entregar formalmente a um cliente)

```bash
node scripts/export-docx.js docs/PROJECT.md
```

Gera um `.docx` no padrão visual Sogest a partir do Markdown — a fonte de
verdade continua sendo o Markdown, nunca o `.docx` editado à parte.
```

- [ ] **Step 4: Criar `docs/decision-records/README.md`**

```markdown
# Decision Records

Um arquivo curto por decisão significativa de governança/método — a versão
deste template do "ADR", sem o prefixo "Architecture" porque aqui a decisão
raramente é técnica.

**Quando criar um:** você tomou uma decisão que alguém poderia questionar
ou desfazer depois sem saber o motivo — um critério de priorização, uma
mudança de cadência, um corte de escopo.

**Formato** (caiba em uma página): `NNNN-titulo-curto.md`
```
# NNNN. Título

**Status:** accepted | superseded by NNNN | deprecated
**Date:** YYYY-MM-DD

## Contexto
O que forçou a decisão.

## Decisão
O que escolhemos, dito diretamente.

## Consequências
O que isso facilita, o que dificulta, no que prestar atenção.
```

Suceder em vez de editar: uma decisão que muda ganha um registro **novo**
marcando o antigo como superseded.

## Índice

_(preencher conforme registros forem criados)_
```

- [ ] **Step 5: Criar `docs/memoria-discursiva.md`**

```markdown
# Memória discursiva

Camada narrativa e relacional — decisões sensíveis, tensões, nuances de
relacionamento, interpretação estratégica que não cabem num board ou numa
issue. **Este arquivo é interno por padrão** (o repositório inteiro já
nasce privado para este tipo de projeto) — separe aqui o que é *compartilhável*
do que é *só nosso*, antes de levar qualquer trecho pra fora.

## Como preencher

- Separe sempre: fato, hipótese, decisão tomada, decisão pendente, ponto
  sensível.
- Não misture com controle operacional granular (isso vive no board).
- Atualize por consolidação (ver `how-we-conduct-a-deliverable.md`), não
  em tempo real.

_(preencher conforme o projeto avança)_
```

- [ ] **Step 6: Criar os `.gitkeep`**

```bash
mkdir -p docs/superpowers/specs docs/superpowers/plans
touch docs/superpowers/specs/.gitkeep docs/superpowers/plans/.gitkeep
```

- [ ] **Step 7: Verificar placeholders**

Run: `grep -l '\[PROJETO\]' docs/PROJECT.md; grep -L '\[TABELA_AMBIENTES\]' docs/PROJECT.md`
Expected: primeira linha lista `docs/PROJECT.md`; segunda linha também lista `docs/PROJECT.md` (confirma que o token de ambientes NÃO está presente, por design)

- [ ] **Step 8: Commit**

```bash
git add docs
git commit -m "docs: add methodology docs skeleton to template"
git push
```

---

### Task 3: `AGENTS.md`, `CONTRIBUTING.md`, `README.md`

**Files:**
- Create: `sogest-methodology-template/AGENTS.md`
- Modify: `sogest-methodology-template/README.md` (já existe da Task 1 — mantém como está, sem mudança)
- Create: `sogest-methodology-template/CONTRIBUTING.md`

**Interfaces:**
- Produces: mesmos placeholders da Task 2.

- [ ] **Step 1: Criar `AGENTS.md`**

```markdown
# [PROJETO] — regras do projeto (leia primeiro)

[DESCRICAO_UMA_LINHA]

Projeto do tipo **Metodologia e Gestão** — condução de projeto e
governança, sem software. Este arquivo é a memória de projeto que todo
assistente de IA deve ler antes de trabalhar aqui. Docs completos:
[`README.md`](README.md), [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Rastreamento do projeto

Trabalho é organizado em **módulos** (`area:` labels) → **epics** (issues +
sub-issues) → **sprints** (milestones), num **board**
(Triage→Backlog→Ready→In Progress→In Review→Done). Guia completo:
[`docs/PROJECT.md`](docs/PROJECT.md).

- **Visão geral do projeto:** rode [`scripts/overview.sh`](scripts/overview.sh).
- **Pegar um ticket:** self-assign + label `status:in-progress` + PR
  rascunho (`Closes #n`) antes de produzir qualquer entregável.

## Módulos deste projeto

[LISTA_MODULOS_COM_EPICS]

## Regras de ouro

1. **Nunca produza o entregável final sem entendimento validado.** Siga o
   método em [`docs/how-we-conduct-a-deliverable.md`](docs/how-we-conduct-a-deliverable.md)
   — entrevista/diagnóstico → arquitetura → gate de aprovação → produção.
2. **Capturar não é atualizar.** Durante a semana, registre fatos numa
   issue de captura; só consolide quando pedido explicitamente.
3. Consulte [`docs/decision-records/`](docs/decision-records/README.md)
   antes de mudar uma decisão já assentada — e adicione um registro novo
   quando tomar uma decisão que alguém possa questionar depois.
4. **Memória discursiva é interna.** Antes de compartilhar qualquer trecho
   de [`docs/memoria-discursiva.md`](docs/memoria-discursiva.md) com um
   cliente, separe o que é compartilhável do que não é.
5. Edite `main` só via PR — sem pipeline de deploy neste tipo de projeto,
   mas o mesmo cuidado de revisão se aplica.
```

- [ ] **Step 2: Criar `CONTRIBUTING.md`**

```markdown
# Contribuindo & Fluxo de trabalho

Como trabalhamos em [PROJETO].

## Acesso

- **Código/conteúdo:** GitHub `pmsartori/[SLUG_PROJETO]` (privado). Peça
  convite de colaborador.
- Este repositório é **privado por padrão** — pode conter informação
  sensível de cliente ou de governança interna.

## Branching & pull requests

Sem pipeline de deploy neste tipo de projeto — fluxo simples:

```
main            sempre a versão vigente; protegida (PR + 1 review)
feature/<nome>  sua branch de trabalho, a partir de main
```

1. `git switch -c feature/descricao-curta`
2. Siga o método antes de produzir qualquer entregável final — ver
   [`docs/how-we-conduct-a-deliverable.md`](docs/how-we-conduct-a-deliverable.md).
3. Abra PR contra `main`. 1 revisor aprova.
4. Squash-merge.

## Rastreamento de trabalho

- Trabalho é rastreado como **GitHub Issues**, organizado no **Project
  board** (Triage → Backlog → Ready → In Progress → In Review → Done).
- Pegue uma issue, self-assign, mova pra *In Progress*, referencie no PR
  (`Closes #NN`).

### Reivindicando trabalho

1. `gh issue edit <n> --add-assignee @me`
2. `gh issue edit <n> --add-label status:in-progress`
3. `gh pr create --draft --fill --base main`
```

- [ ] **Step 3: Verificar placeholders**

Run: `grep -l '\[PROJETO\]' AGENTS.md CONTRIBUTING.md`
Expected: lista os 2 arquivos

- [ ] **Step 4: Commit**

```bash
git add AGENTS.md CONTRIBUTING.md
git commit -m "docs: add AGENTS.md and CONTRIBUTING.md for methodology type"
git push
```

---

### Task 4: `templates/` e `.github/ISSUE_TEMPLATE/`

**Files:**
- Create: `sogest-methodology-template/templates/master-briefing-template.md`
- Create: `sogest-methodology-template/templates/weekly-report-template.md`
- Create: `sogest-methodology-template/.github/ISSUE_TEMPLATE/gap.yml`
- Create: `sogest-methodology-template/.github/ISSUE_TEMPLATE/suggestion.yml`
- Create: `sogest-methodology-template/.github/ISSUE_TEMPLATE/config.yml`

- [ ] **Step 1: Criar `templates/master-briefing-template.md`**

```markdown
# Master Briefing — [PROJETO]

_Documento-mestre de contexto, tese de atuação, histórico, governança,
riscos e prioridades. Preencha via `superpowers:brainstorming` — não pule
pra aqui sem passar pelo método (ver `docs/how-we-conduct-a-deliverable.md`)._

## Resumo executivo

## Ficha técnica

## Modelo de negócio / contexto

## Histórico do mandato

## Escopo atual

## Arquitetura estratégica (objetivos, gates, portfólio)

## Riscos e pontos sensíveis

## Governança, cadência e rituais

## Critérios de sucesso

## O que fazer e o que evitar

## Fontes utilizadas

## Pendências e lacunas
```

- [ ] **Step 2: Criar `templates/weekly-report-template.md`**

```markdown
# Weekly Report — [PROJETO] — semana de {{DATA}}

_Preenchido na Consolidação Semanal (ver
`docs/how-we-conduct-a-deliverable.md`) — nunca durante a captura contínua._

## O que avançou

## O que travou

## Decisões tomadas

## Decisões pendentes

## Próxima semana
```

- [ ] **Step 3: Criar `.github/ISSUE_TEMPLATE/gap.yml`**

```yaml
name: 🔍 Lacuna ou risco
description: Registrar uma lacuna de informação, risco ou ponto sensível encontrado na condução
labels: ["gap", "needs-triage"]
body:
  - type: textarea
    id: description
    attributes:
      label: O que foi encontrado?
    validations:
      required: true
  - type: textarea
    id: impact
    attributes:
      label: Impacto se não for resolvido
    validations:
      required: false
```

- [ ] **Step 4: Criar `.github/ISSUE_TEMPLATE/suggestion.yml`**

```yaml
name: 💡 Sugestão de frente
description: Propor uma frente de trabalho nova — cai em Triage
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

- [ ] **Step 5: Criar `.github/ISSUE_TEMPLATE/config.yml`**

```yaml
blank_issues_enabled: true
contact_links:
  - name: 📋 Como conduzimos
    url: https://github.com/pmsartori/[SLUG_PROJETO]/blob/main/docs/PROJECT.md
    about: A estrutura de módulos, epics, sprints, board e o método de condução.
```

- [ ] **Step 6: Commit**

```bash
git add templates .github
git commit -m "feat: add deliverable templates and issue templates"
git push
```

---

### Task 5: `scripts/overview.sh` e `scripts/export-docx.js` (com testes)

**Files:**
- Create: `sogest-methodology-template/scripts/overview.sh`
- Create: `sogest-methodology-template/scripts/export-docx.js`
- Create: `sogest-methodology-template/scripts/export-docx.test.js`
- Create: `sogest-methodology-template/package.json`

**Interfaces:**
- Produces: `classifyLine(line)` — função pura exportada por `export-docx.js`,
  usada pelos testes e pelo gerador. Retorna um objeto `{type, ...}` onde
  `type` é um de: `heading1`, `heading2`, `heading3`, `bullet`,
  `table-row`, `table-separator`, `blank`, `paragraph`.

- [ ] **Step 1: Criar `scripts/overview.sh`** (idêntico ao do template de Software)

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

- [ ] **Step 2: Checar sintaxe e dar permissão de execução**

Run: `bash -n scripts/overview.sh && chmod +x scripts/overview.sh && echo ok`
Expected: `ok`

- [ ] **Step 3: Criar `package.json`**

```json
{
  "name": "sogest-methodology-template",
  "private": true,
  "type": "commonjs",
  "dependencies": {
    "docx": "^9.0.0"
  }
}
```

- [ ] **Step 4: Escrever o teste primeiro — `scripts/export-docx.test.js`**

```javascript
const { test } = require('node:test');
const assert = require('node:assert/strict');
const { classifyLine } = require('./export-docx.js');

test('classifies heading levels', () => {
  assert.deepEqual(classifyLine('# Título'), { type: 'heading1', text: 'Título' });
  assert.deepEqual(classifyLine('## Sub'), { type: 'heading2', text: 'Sub' });
  assert.deepEqual(classifyLine('### Sub-sub'), { type: 'heading3', text: 'Sub-sub' });
});

test('classifies bullets', () => {
  assert.deepEqual(classifyLine('- item um'), { type: 'bullet', text: 'item um' });
});

test('classifies table separator rows distinctly from table rows', () => {
  assert.equal(classifyLine('|---|---|').type, 'table-separator');
  assert.equal(classifyLine('| a | b |').type, 'table-row');
});

test('splits table row cells trimmed, ignoring outer pipes', () => {
  const row = classifyLine('| Coluna A | Coluna B |');
  assert.deepEqual(row.cells, ['Coluna A', 'Coluna B']);
});

test('classifies blank lines', () => {
  assert.equal(classifyLine('').type, 'blank');
  assert.equal(classifyLine('   ').type, 'blank');
});

test('classifies anything else as a paragraph', () => {
  assert.deepEqual(classifyLine('Texto qualquer.'), { type: 'paragraph', text: 'Texto qualquer.' });
});
```

- [ ] **Step 5: Rodar os testes e confirmar que falham (RED)**

Run: `node --test scripts/export-docx.test.js`
Expected: FAIL — `Cannot find module './export-docx.js'` ou `classifyLine is not a function` (o arquivo ainda não existe)

- [ ] **Step 6: Implementar `scripts/export-docx.js`**

```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, WidthType, HeadingLevel, ShadingType } = require('docx');
const fs = require('fs');
const path = require('path');

const PRIMARY = "1A365D";
const WHITE = "FFFFFF";

function classifyLine(line) {
  const trimmed = line.trim();
  if (trimmed === '') return { type: 'blank' };
  if (/^#{1,3}\s+/.test(trimmed)) {
    const level = trimmed.match(/^(#{1,3})/)[1].length;
    return { type: `heading${level}`, text: trimmed.replace(/^#{1,3}\s+/, '') };
  }
  if (/^-\s+/.test(trimmed)) {
    return { type: 'bullet', text: trimmed.replace(/^-\s+/, '') };
  }
  if (/^\|?[\s:|-]+\|?$/.test(trimmed) && trimmed.includes('-')) {
    return { type: 'table-separator' };
  }
  if (/^\|.*\|$/.test(trimmed)) {
    const cells = trimmed.slice(1, -1).split('|').map(c => c.trim());
    return { type: 'table-row', cells };
  }
  return { type: 'paragraph', text: trimmed };
}

function headingParagraph(text, level) {
  const headingLevel = { 1: HeadingLevel.HEADING_1, 2: HeadingLevel.HEADING_2, 3: HeadingLevel.HEADING_3 }[level];
  return new Paragraph({ heading: headingLevel, spacing: { before: 300, after: 100 }, children: [new TextRun({ text, bold: true, color: PRIMARY })] });
}

function paraParagraph(text) {
  return new Paragraph({ spacing: { after: 120 }, children: [new TextRun({ text, size: 22 })] });
}

function bulletParagraph(text) {
  return new Paragraph({ bullet: { level: 0 }, spacing: { after: 60 }, children: [new TextRun({ text, size: 22 })] });
}

function tableRowEl(cells, isHeader) {
  return new TableRow({
    children: cells.map(text => new TableCell({
      width: { size: 100 / cells.length, type: WidthType.PERCENTAGE },
      shading: isHeader ? { type: ShadingType.SOLID, color: PRIMARY } : undefined,
      children: [new Paragraph({ children: [new TextRun({ text, size: 20, bold: isHeader, color: isHeader ? WHITE : "333333" })] })]
    }))
  });
}

function buildDocChildren(markdown) {
  const lines = markdown.split('\n');
  const children = [];
  let tableRows = null;

  const flushTable = () => {
    if (tableRows && tableRows.length) {
      children.push(new Table({ width: { size: 100, type: WidthType.PERCENTAGE }, rows: tableRows }));
    }
    tableRows = null;
  };

  for (const line of lines) {
    const c = classifyLine(line);
    if (c.type === 'table-row') {
      if (!tableRows) tableRows = [];
      tableRows.push(tableRowEl(c.cells, tableRows.length === 0));
      continue;
    }
    if (c.type === 'table-separator') continue; // marks header/body boundary, not rendered
    flushTable();
    if (c.type === 'heading1') children.push(headingParagraph(c.text, 1));
    else if (c.type === 'heading2') children.push(headingParagraph(c.text, 2));
    else if (c.type === 'heading3') children.push(headingParagraph(c.text, 3));
    else if (c.type === 'bullet') children.push(bulletParagraph(c.text));
    else if (c.type === 'paragraph') children.push(paraParagraph(c.text));
    // blank: skip
  }
  flushTable();
  return children;
}

async function exportDocx(mdPath) {
  const markdown = fs.readFileSync(mdPath, 'utf-8');
  const doc = new Document({
    styles: { default: { document: { run: { font: "Calibri", size: 22 } } } },
    sections: [{ properties: { page: { margin: { top: 1000, bottom: 1000, left: 1200, right: 1200 } } }, children: buildDocChildren(markdown) }]
  });
  const buffer = await Packer.toBuffer(doc);
  const outPath = mdPath.replace(/\.md$/, '.docx');
  fs.writeFileSync(outPath, buffer);
  console.log(`Written: ${outPath}`);
}

module.exports = { classifyLine, buildDocChildren };

if (require.main === module) {
  const mdPath = process.argv[2];
  if (!mdPath) {
    console.error('Uso: node scripts/export-docx.js <arquivo.md>');
    process.exit(1);
  }
  exportDocx(path.resolve(mdPath)).catch(err => { console.error(err); process.exit(1); });
}
```

- [ ] **Step 7: Rodar os testes e confirmar que passam (GREEN)**

Run: `node --test scripts/export-docx.test.js`
Expected: PASS — 6/6 testes

- [ ] **Step 8: Commit**

```bash
git add scripts package.json
git commit -m "feat: add overview.sh and export-docx.js (Markdown to Word) with tests"
git push
```

---

### Task 6: Skill `new-project` — Fase 1 ramifica por tipo

**Files:**
- Modify: `sogest-ai-project-method/skills/new-project/SKILL.md`

**Interfaces:**
- Consumes: `pmsartori/sogest-methodology-template` publicado (Tasks 1–5).

Antes de come al implementador: confirme com `grep -n "^[0-9]\+\. \*\*" skills/new-project/SKILL.md` que a Fase 1 tem exatamente os 7 itens já conhecidos (Tipo de projeto / Nome / Privado-público / Módulos / Serviços locais / Modelo de deploy / Cadência de sprint) antes de editar — o arquivo já foi modificado por um plano anterior (Docker) e a numeração precisa bater.

- [ ] **Step 1: Reescrever o item 1 da Fase 1** (remove o "pare", já ramifica de verdade)

Troque o texto atual do item 1:

```markdown
1. **Tipo de projeto:** Software ou Metodologia e Gestão? (Metodologia e
   Gestão está fora do escopo desta versão da skill — se escolhido, avise
   que ainda não está implementado e pare.)
```

por:

```markdown
1. **Tipo de projeto:** Software ou Metodologia e Gestão? A resposta
   define qual template usar (`sogest-project-template` ou
   `sogest-methodology-template`) e quais perguntas seguintes se aplicam —
   itens marcados "(Software)" abaixo só são perguntados se este tipo foi
   escolhido.
```

- [ ] **Step 2: Marcar o item de privacidade como condicional na recomendação**

Troque:

```markdown
3. **Privado ou público?** (recomendação padrão: público, para tipo
   Software.)
```

por:

```markdown
3. **Privado ou público?** (recomendação padrão: **público** para tipo
   Software; **privado** para tipo Metodologia e Gestão — esses projetos
   costumam conter informação sensível de cliente.)
```

- [ ] **Step 3: Marcar os itens de Docker e deploy como exclusivos de Software**

Troque o início do item de serviços locais de:

```markdown
5. **Este projeto precisa de serviços locais** (banco de dados, cache)?
```

por:

```markdown
5. **(Software) Este projeto precisa de serviços locais** (banco de dados, cache)?
```

E troque o início do item de modelo de deploy de:

```markdown
6. **Modelo de deploy:** SSH próprio / Vercel-Netlify / Nenhum (manual).
```

por:

```markdown
6. **(Software) Modelo de deploy:** SSH próprio / Vercel-Netlify / Nenhum (manual).
```

Não altere mais nada no item 7 (Cadência de sprint) — ele se aplica aos
dois tipos, sem mudança.

- [ ] **Step 4: Validar frontmatter ainda íntegro**

Run: `node -e "const fs=require('fs'); const c=fs.readFileSync('skills/new-project/SKILL.md','utf-8'); console.log('frontmatter ok:', c.startsWith('---\nname: new-project'))"`
Expected: `frontmatter ok: true`

- [ ] **Step 5: Commit**

```bash
git add skills/new-project/SKILL.md
git commit -m "feat: branch Fase 1 questions by project type (Software vs Metodologia e Gestão)"
git push
```

---

### Task 7: Skill `new-project` — Fase 2 ramifica por tipo

**Files:**
- Modify: `sogest-ai-project-method/skills/new-project/SKILL.md`
- Create: `sogest-ai-project-method/skills/new-project/references/methodology-type.md`

**Interfaces:**
- Consumes: Task 6 (Fase 1 já ramifica); templates dos dois tipos já publicados.

Antes de começar: confirme com `grep -n "^[0-9]\+\. \*\*" skills/new-project/SKILL.md` que a Fase 2 tem exatamente os 11 itens já conhecidos (Criar repo / Substituir placeholders / Aplicar deploy / Docker / Commit main / develop-test-protect / labels / epic / milestone / board / resumo final).

- [ ] **Step 1: Item 1 — escolher o template certo**

Troque:

```markdown
1. **Criar o repositório a partir do template:**
   ```bash
   gh repo create pmsartori/<slug> --template pmsartori/sogest-project-template --public   # ou --private
   for i in $(seq 1 10); do
     gh api repos/pmsartori/<slug>/contents/AGENTS.md >/dev/null 2>&1 && break
     sleep 2
   done
   git clone https://github.com/pmsartori/<slug>.git
   cd <slug>
   ```
```

por:

```markdown
1. **Criar o repositório a partir do template certo** (depende do tipo
   escolhido na Fase 1 — Software usa `sogest-project-template`,
   Metodologia e Gestão usa `sogest-methodology-template`):
   ```bash
   TEMPLATE=pmsartori/sogest-project-template        # se tipo = Software
   TEMPLATE=pmsartori/sogest-methodology-template     # se tipo = Metodologia e Gestão
   gh repo create pmsartori/<slug> --template "$TEMPLATE" --public   # ou --private
   for i in $(seq 1 10); do
     gh api repos/pmsartori/<slug>/contents/AGENTS.md >/dev/null 2>&1 && break
     sleep 2
   done
   git clone https://github.com/pmsartori/<slug>.git
   cd <slug>
   ```
```

- [ ] **Step 2: Item 2 — remover a menção a `[TABELA_AMBIENTES]` quando o tipo é Metodologia**

Localize o item 2 ("Substituir os placeholders...") e acrescente, logo após
a lista de placeholders existente, esta nota:

```markdown
   **(Metodologia e Gestão)** este tipo não tem o placeholder
   `[TABELA_AMBIENTES]` em nenhum arquivo — pule esse item da lista.
```

- [ ] **Step 3: Itens 3 e 4 (deploy e Docker) — marcar como exclusivos de Software**

Troque o início do item 3 de "**Aplicar o modelo de deploy escolhido**"
para "**(Software) Aplicar o modelo de deploy escolhido**", e o início do
item 4 de "**Se a resposta da Fase 1 foi \"sim\" para serviços locais**"
para "**(Software) Se a resposta da Fase 1 foi \"sim\" para serviços
locais**". Não altere o conteúdo dos blocos de código desses dois itens.

- [ ] **Step 4: Item 6 (branches) — pular develop/test para Metodologia**

Troque o início do item 6 (branches) de:

```markdown
6. **Criar `develop` e `test` a partir de `main`, e proteger `main`:**
```

por:

```markdown
6. **Proteger `main`** (e, **apenas para tipo Software**, criar `develop`
   e `test` a partir dela primeiro — Metodologia e Gestão não tem pipeline
   de deploy, protege só `main`):
   ```bash
   # (Software) pule este bloco develop/test se o tipo for Metodologia e Gestão
   git checkout -b develop
   git push -u origin develop
   git checkout -b test
   git push -u origin test
   git checkout main
   ```
```

Mantenha o restante do item 6 (o bloco do `gh api ... branches/main/protection`
e a nota sobre 403 em plano Free) exatamente como está — ele se aplica aos
dois tipos. Ajuste só a última frase do item, que hoje diz "Termine este
passo na branch `develop`" — troque por: "**(Software)** termine este
passo na branch `develop`. **(Metodologia e Gestão)** termine na própria
`main`, já que não existe `develop` neste tipo."

- [ ] **Step 5: Item 11 (resumo final) — ajustar os próximos passos por tipo**

No item 11 ("Resumo final"), acrescente à lista de próximos passos
manuais: "**(Metodologia e Gestão)** para exportar um documento em Word no
padrão Sogest, rode `node scripts/export-docx.js <arquivo.md>`."

- [ ] **Step 6: Criar `skills/new-project/references/methodology-type.md`**

```markdown
# Tipo de projeto: Metodologia e Gestão

Checklist rápido do que muda em relação ao tipo Software:

- Template: `pmsartori/sogest-methodology-template`, não
  `sogest-project-template`.
- Recomendação padrão de visibilidade: **privado**.
- Sem perguntas de Docker/deploy na Fase 1.
- Sem placeholder `[TABELA_AMBIENTES]`.
- Sem branches `develop`/`test` — só `main`, protegida.
- Resumo final menciona `scripts/export-docx.js` como próximo passo
  opcional (Markdown → Word no padrão visual Sogest).
```

- [ ] **Step 7: Validar frontmatter ainda íntegro**

Run: `node -e "const fs=require('fs'); const c=fs.readFileSync('skills/new-project/SKILL.md','utf-8'); console.log('frontmatter ok:', c.startsWith('---\nname: new-project'))"`
Expected: `frontmatter ok: true`

- [ ] **Step 8: Commit**

```bash
git add skills/new-project/SKILL.md skills/new-project/references/methodology-type.md
git commit -m "feat: branch Fase 2 automation by project type (Software vs Metodologia e Gestão)"
git push
```

---

### Task 8: Teste ponta a ponta do tipo Metodologia e Gestão (dry run real)

**Files:** nenhum arquivo novo — este task só executa e verifica.

**Interfaces:**
- Consumes: Tasks 1–7.

- [ ] **Step 1: Criar um repositório de teste do tipo Metodologia e Gestão**

Siga manualmente o fluxo da Fase 2 (item por item, como um operador real
seguiria a skill) contra `pmsartori/sogest-method-e2e-methodology-test`,
respondendo: tipo = Metodologia e Gestão, privado, módulo único "core",
sprint de 7 dias.

- [ ] **Step 2: Verificar o resultado**

Run: `gh repo view pmsartori/sogest-method-e2e-methodology-test --json isPrivate,name`
Expected: `isPrivate: true`

Run: `gh api repos/pmsartori/sogest-method-e2e-methodology-test/contents/docs/how-we-conduct-a-deliverable.md --jq .name`
Expected: `how-we-conduct-a-deliverable.md`

Run: `gh api repos/pmsartori/sogest-method-e2e-methodology-test/contents/docs/DEPLOYMENT.md 2>&1`
Expected: 404 (este tipo não tem `DEPLOYMENT.md` — nunca existiu no template, nada a remover)

Run: `gh api repos/pmsartori/sogest-method-e2e-methodology-test/branches --jq '.[].name'`
Expected: só `main` (sem `develop`/`test`)

Run: `gh api repos/pmsartori/sogest-method-e2e-methodology-test/branches/main --jq .protected`
Expected: `true`

Run: `gh issue list --repo pmsartori/sogest-method-e2e-methodology-test --label epic --json title`
Expected: `[{"title":"[EPIC] core"}]`

- [ ] **Step 3: Testar `export-docx.js` de verdade**

```bash
git clone https://github.com/pmsartori/sogest-method-e2e-methodology-test.git /tmp-export-test 2>&1 || git clone https://github.com/pmsartori/sogest-method-e2e-methodology-test.git "$HOME/export-test"
cd "$HOME/export-test" && npm install --no-fund --no-audit && node scripts/export-docx.js docs/PROJECT.md
```

Run: `test -f "$HOME/export-test/docs/PROJECT.docx" && echo OK`
Expected: `OK`

- [ ] **Step 4: Limpar**

```bash
rm -rf "$HOME/export-test"
gh project list --owner pmsartori --format json -q '.projects[] | select(.title | contains("e2e-methodology")) | .number'
# para cada número retornado: gh project delete <numero> --owner pmsartori
```

Peça ao usuário para apagar `pmsartori/sogest-method-e2e-methodology-test`
manualmente (mesma limitação de escopo `delete_repo` do dry run anterior),
a menos que o escopo já tenha sido concedido nesta sessão.

- [ ] **Step 5: Reportar resultado**

Resuma o que passou/falhou antes de considerar a Task 8 concluída. Se
algum bug real for encontrado (como no dry run do tipo Software), registre
os achados exatos e trate como uma correção adicional antes de fechar o
plano — não relate sucesso com um bug conhecido em aberto.
