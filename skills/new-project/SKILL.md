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
