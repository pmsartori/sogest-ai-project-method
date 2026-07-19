# Docker Local Services (tipo Software) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Adicionar suporte opcional a serviços locais via Docker Compose ao template de Software e à skill `new-project`, sem impor a dependência a projetos que não precisam.

**Architecture:** Uma pasta descartável `optional/` em `sogest-project-template` (mesmo padrão de `variants/`), copiada condicionalmente pela Fase 2 da skill com base numa nova pergunta da Fase 1, e sempre removida no final — independente da resposta.

**Tech Stack:** Docker Compose (YAML), Bash (skill automation via `gh`/`git`/`docker`), Markdown.

## Global Constraints

- A skill **nunca** roda `docker compose up` sozinha — só escreve o arquivo e documenta o comando no resumo final.
- A pasta `optional/` é apagada em TODOS os casos (resposta sim ou não) antes do commit — nada de descartável pode sobrar no repositório final.
- Credenciais no `docker-compose.yml` são fixas e de ambiente local (`dev`/`localdev`), nunca segredo real — consistente com o `.env.example` já usado no Prisma.
- Se `docker info` falhar (Docker não instalado/rodando), a skill instrui e **pausa** — mesma disciplina das checagens de pré-voo existentes.

---

## File Structure

```
sogest-project-template/
└── optional/
    ├── docker-compose.yml                        NOVO
    └── CONTRIBUTING-local-services-snippet.md     NOVO

sogest-ai-project-method/
└── skills/new-project/SKILL.md                     MODIFICADO (Fase 1 + Fase 2)
```

---

### Task 1: Arquivos opcionais de serviços locais (`sogest-project-template`)

**Files:**
- Create: `sogest-project-template/optional/docker-compose.yml`
- Create: `sogest-project-template/optional/CONTRIBUTING-local-services-snippet.md`

**Interfaces:**
- Produces: os dois arquivos que a Task 2 (Fase 2 da skill) copia/anexa condicionalmente. O placeholder `[SLUG_PROJETO]` já é o mesmo usado em todo o resto do template — nenhum placeholder novo é introduzido.

- [ ] **Step 1: Criar `optional/docker-compose.yml`**

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

- [ ] **Step 2: Criar `optional/CONTRIBUTING-local-services-snippet.md`**

```markdown

## Ambiente local

Este projeto usa Docker Compose para serviços locais (banco de dados).

```bash
docker compose up -d      # sobe o Postgres local
docker compose down       # derruba
docker compose down -v    # derruba e apaga os dados (reset completo)
```

Credenciais padrão (ambiente local, não são segredo): usuário `dev`, senha
`localdev`, banco `[SLUG_PROJETO]`, porta `5432`. Ajuste `docker-compose.yml`
se precisar de outro serviço (Redis, etc.) — há um comentário indicando
onde adicionar.
```

- [ ] **Step 3: Verificar que o placeholder existe nos dois arquivos**

Run: `grep -l '\[SLUG_PROJETO\]' optional/docker-compose.yml optional/CONTRIBUTING-local-services-snippet.md`
Expected: lista os 2 arquivos

- [ ] **Step 4: Commit**

```bash
git add optional
git commit -m "feat: add optional Docker local-services scaffold to template"
git push
```

---

### Task 2: Skill `new-project` — pergunta condicional + aplicação (Fase 1 e Fase 2)

**Files:**
- Modify: `sogest-ai-project-method/skills/new-project/SKILL.md`

**Interfaces:**
- Consumes: `optional/docker-compose.yml` e `optional/CONTRIBUTING-local-services-snippet.md` de `sogest-project-template` (Task 1).
- Produces: o fluxo completo que a Task 3 (dry run) exercita.

- [ ] **Step 1: Adicionar a pergunta na Fase 1**

Localize, dentro de "## Fase 1 — Perguntas", o item 4 ("Módulos do
projeto…") e o item 5 ("Modelo de deploy…"). Insira um novo item **entre
os dois**, renumerando o item 5 (modelo de deploy) para 6 e o item 6
(cadência de sprint) para 7:

```markdown
5. **Este projeto precisa de serviços locais** (banco de dados, cache)?
   (sim/não)
   - Se sim: antes de prosseguir, rode `docker info`. Se falhar, instrua a
     instalar/abrir o Docker Desktop e **pare** — mesma disciplina das
     checagens de pré-voo (nunca prossiga com uma dependência ausente sem
     avisar).
```

- [ ] **Step 2: Adicionar o passo condicional na Fase 2**

Localize, dentro de "## Fase 2 — Automação", o passo "3. **Aplicar o
modelo de deploy escolhido**" e o passo seguinte ("4. **Commit e push em
`main`**"). Insira um novo passo **entre os dois**, renumerando todos os
passos seguintes em +1 (o passo 4 atual vira 5, o 5 vira 6, e assim por
diante até o resumo final):

```markdown
4. **Se a resposta da Fase 1 foi "sim" para serviços locais**, aplicar o
   scaffold de Docker:
   ```bash
   cp optional/docker-compose.yml docker-compose.yml
   sed -i "s/\[SLUG_PROJETO\]/<slug>/g" docker-compose.yml
   cat optional/CONTRIBUTING-local-services-snippet.md >> CONTRIBUTING.md
   ```
   **Em qualquer caso** (sim ou não), remova a pasta descartável antes do
   commit:
   ```bash
   rm -rf optional
   ```
```

- [ ] **Step 3: Atualizar o resumo final (último passo da Fase 2)**

No passo final ("Resumo final"), acrescente à lista de próximos passos
manuais: "se o projeto usa serviços locais, rode `docker compose up -d`
antes de começar a desenvolver."

- [ ] **Step 4: Validar frontmatter/estrutura do arquivo ainda íntegra**

Run: `node -e "const fs=require('fs'); const c=fs.readFileSync('skills/new-project/SKILL.md','utf-8'); const m=c.match(/^---\n([\s\S]*?)\n---/); console.log('frontmatter ok:', !!m && c.includes('name: new-project'))"`
Expected: `frontmatter ok: true`

- [ ] **Step 5: Commit**

```bash
git add skills/new-project/SKILL.md
git commit -m "feat: add optional Docker local-services question and application step"
git push
```

---

### Task 3: Dry run do novo caminho condicional

**Files:** nenhum arquivo novo — este task só executa e verifica.

**Interfaces:**
- Consumes: Tasks 1 e 2.

Este dry run não precisa criar um repositório GitHub novo — os mecanismos
específicos do GitHub (template assíncrono, branch protection, labels,
board) já foram provados no dry run do plano anterior e não são afetados
por esta mudança. Basta verificar, num clone local do template, que a
lógica de cópia condicional + limpeza funciona corretamente nos dois
ramos (sim e não).

- [ ] **Step 1: Clonar o template numa pasta de teste**

Use uma pasta local de verdade, não `/tmp` — em Git Bash no Windows,
`/tmp` frequentemente não é gravável (já bateu nisso no dry run do plano
anterior). Use `$HOME` ou o diretório de scratch da sessão:

```bash
mkdir -p "$HOME/docker-e2e-test" && cd "$HOME/docker-e2e-test"
git clone https://github.com/pmsartori/sogest-project-template.git ramo-sim
git clone https://github.com/pmsartori/sogest-project-template.git ramo-nao
```

- [ ] **Step 2: Simular o ramo "sim" (precisa de serviços locais)**

```bash
cd "$HOME/docker-e2e-test/ramo-sim"
cp optional/docker-compose.yml docker-compose.yml
sed -i "s/\[SLUG_PROJETO\]/docker-e2e-test/g" docker-compose.yml
cat optional/CONTRIBUTING-local-services-snippet.md >> CONTRIBUTING.md
rm -rf optional
```

- [ ] **Step 3: Verificar o ramo "sim"**

Run: `test -f docker-compose.yml && grep -q "docker-e2e-test-pg-local" docker-compose.yml && grep -q "Ambiente local" CONTRIBUTING.md && ! test -d optional && echo OK`
Expected: `OK`

- [ ] **Step 4: Simular e verificar o ramo "não" (não precisa de serviços locais)**

```bash
cd "$HOME/docker-e2e-test/ramo-nao"
rm -rf optional
```

Run: `test ! -f docker-compose.yml && ! grep -q "Ambiente local" CONTRIBUTING.md && ! test -d optional && echo OK`
Expected: `OK`

- [ ] **Step 5: Limpar a pasta de teste**

```bash
cd "$HOME" && rm -rf "$HOME/docker-e2e-test"
```

- [ ] **Step 6: Reportar resultado**

Resuma se os dois ramos passaram antes de considerar a Task 3 concluída.
