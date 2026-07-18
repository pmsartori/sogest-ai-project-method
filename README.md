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
