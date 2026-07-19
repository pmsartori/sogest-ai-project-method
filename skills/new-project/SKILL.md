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

## Fase 1 — Perguntas

Pergunte uma de cada vez.

1. **Tipo de projeto:** Software ou Metodologia e Gestão? A resposta
   define qual template usar (`sogest-project-template` ou
   `sogest-methodology-template`) e quais perguntas seguintes se aplicam —
   itens marcados "(Software)" abaixo só são perguntados se este tipo foi
   escolhido.
2. **Nome do projeto** (vira o slug do repo, kebab-case) + descrição de
   uma linha.
3. **Privado ou público?** (recomendação padrão: **público** para tipo
   Software; **privado** para tipo Metodologia e Gestão — esses projetos
   costumam conter informação sensível de cliente.)
4. **Módulos do projeto** — lista livre, ex: "Tenancy, Reporting, Import".
   Cada um vira uma label `area:<slug>` + 1 epic.
5. **(Software) Este projeto precisa de serviços locais** (banco de dados, cache)?
   (sim/não)
   - Se sim: antes de prosseguir, rode `docker info`. Se falhar, instrua a
     instalar/abrir o Docker Desktop e **pare** — mesma disciplina das
     checagens de pré-voo (nunca prossiga com uma dependência ausente sem
     avisar).
6. **(Software) Modelo de deploy:** SSH próprio / Vercel-Netlify / Nenhum (manual).
   - Se SSH: pergunte host, usuário, caminho remoto. As credenciais reais
     (chave SSH) ficam como próximo passo manual — nunca peça a chave em
     texto no chat.
7. **Cadência de sprint** (ex.: 2 semanas) → define a data de vencimento
   do milestone "Sprint 1" (hoje + N dias).

Confirme um resumo das respostas antes de prosseguir para a Fase 2.

## Fase 2 — Automação

Execute nesta ordem exata. Pare e reporte se qualquer comando falhar.

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

   **(Metodologia e Gestão)** este tipo não tem o placeholder
   `[TABELA_AMBIENTES]` em nenhum arquivo — pule esse item da lista.

3. **(Software) Aplicar o modelo de deploy escolhido** (consulte
   `references/deploy-<modelo>.md` para o passo a passo exato):
   ```bash
   cp variants/<modelo>/DEPLOYMENT.md docs/DEPLOYMENT.md
   mkdir -p .github/workflows
   cp variants/<modelo>/workflows/*.yml .github/workflows/ 2>/dev/null || true
   rm -rf variants
   ```

4. **(Software) Se a resposta da Fase 1 foi "sim" para serviços
   locais**, aplicar o scaffold de Docker:
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

5. **Commit e push em `main`** (antes de criar `develop`/`test`, para que
   as duas nasçam já com o conteúdo preenchido, não com os placeholders
   crus do template):
   ```bash
   git add -A
   git commit -m "chore: fill onboarding placeholders"
   git push
   ```

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
   ```bash
   cat > /tmp/branch-protection.json <<'JSON'
   {
     "required_status_checks": null,
     "enforce_admins": false,
     "required_pull_request_reviews": {"required_approving_review_count": 1},
     "restrictions": null
   }
   JSON
   gh api -X PUT repos/pmsartori/<slug>/branches/main/protection --input /tmp/branch-protection.json
   # (Software) git checkout develop — (Metodologia e Gestão) não execute
   # este checkout, permaneça em main, pois develop não existe neste tipo
   ```
   Se o repositório for privado num plano GitHub Free, a API de branch
   protection pode retornar 403 (recurso de plano pago) — nesse caso,
   avise o usuário e trate como próximo passo manual em vez de falhar o
   fluxo inteiro. **(Software)** termine este passo na branch `develop`
   (rode o `git checkout develop` comentado acima). **(Metodologia e
   Gestão)** termine na própria `main`, já que não existe `develop` neste
   tipo — não rode o `git checkout develop`.

7. **Criar as labels:**
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

8. **Criar 1 epic por módulo:**
   ```bash
   for m in "${MODULOS[@]}"; do
     gh issue create --title "[EPIC] $m" --label "epic,area:$m" --body "Epic do módulo $m."
   done
   ```

9. **Criar o milestone da Sprint 1:**
   ```bash
   gh api repos/pmsartori/<slug>/milestones -f title="Sprint 1" -f due_on="<hoje+N_dias>T00:00:00Z"
   ```

10. **Criar o board:**
   ```bash
   OWNER=pmsartori REPO=pmsartori/<slug> TITLE="<Projeto> Roadmap" bash scripts/setup-board.sh
   ```

11. **Resumo final** — imprima para o usuário: link do repo, link do
    board, link dos milestones, e a lista de próximos passos manuais (ex.:
    "configure os secrets SSH_HOST/SSH_USER/SSH_KEY/DEPLOY_PATH antes do
    primeiro deploy" se o modelo escolhido foi SSH; branch protection
    manual se a API retornou 403; se o projeto usa serviços locais, rode
    `docker compose up -d` antes de começar a desenvolver). **(Metodologia
    e Gestão)** para exportar um documento em Word no padrão Sogest, rode
    `node scripts/export-docx.js <arquivo.md>`.
