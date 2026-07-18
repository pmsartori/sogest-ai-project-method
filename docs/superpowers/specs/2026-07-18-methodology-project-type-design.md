# Sogest AI Project Method — tipo "Metodologia e Gestão" (sem software)

**Status:** accepted
**Date:** 2026-07-18

## Contexto

O design anterior (`2026-07-18-sogest-ai-project-method-design.md`) cobre
apenas projetos de software: repo com código, CI, pipeline de deploy em 3
ambientes. Mas boa parte do trabalho real da Sogest não produz software —
produz **condução de projeto e governança**: estruturar a gestão de um
cliente, desenhar um sistema de gestão interno, formalizar um método. Hoje
isso é feito via um OPS (Operational Process Specification) rodado num chat
de IA solto, que gera Word/Excel no final
(`20260705_OPS_SOGEST_Master_Build_Client_Project_Management_System_PT_v1.2`),
sem rastreabilidade em git, sem board, sem tracking colaborativo.

O gatilho concreto é o **"Sistema de Gestão da Sogest"** — o projeto-mãe que
vai formalizar como a Sogest usa IA, automação e software em todas as
esferas (gestão de pessoas, autogestão, entrega a cliente, etc.). Esse tipo
de projeto (condução/metodologia, não software) vai se repetir — por isso
vira uma segunda modalidade do instalador, não um projeto único ad hoc.

## Decisão

O instalador `sogest-ai-project-method` passa a suportar dois tipos de
projeto, escolhidos na primeira pergunta do onboarding: **Software** ou
**Metodologia e Gestão**. A escolha determina qual repositório-template é
usado; todo o resto do fluxo (pré-voo, labels, board, milestones, módulos →
epics) é compartilhado entre os dois tipos, pois já era genérico.

```
pmsartori/sogest-project-template        Software (já existe)
pmsartori/sogest-methodology-template     Metodologia e Gestão (novo)
```

### Estrutura do novo template

```
sogest-methodology-template/
├── AGENTS.md                       regras pro assistente + disciplina de gates (ver "O método")
├── CONTRIBUTING.md                  ritual de claim/PR — sem seção de ambientes/deploy
├── README.md
├── docs/
│   ├── PROJECT.md                   manual — mesmas camadas do template de software (código→
│   │                                 tracking→método), com a camada "git flow/deploy" substituída
│   │                                 por "cadência de governança"
│   ├── glossary.md
│   ├── how-we-conduct-a-deliverable.md   o método (ver abaixo)
│   ├── decision-records/            equivalente ao ADR, sem o prefixo "Architecture"
│   │   └── README.md                mesmo formato Contexto/Decisão/Consequências; mesma regra
│   │                                 de suceder em vez de editar
│   ├── memoria-discursiva.md        camada narrativa/relacional, sensível por padrão
│   └── superpowers/{specs,plans}/
├── templates/
│   ├── master-briefing-template.md
│   └── weekly-report-template.md
├── scripts/
│   ├── overview.sh                  reaproveitado do template de software, já é genérico
│   └── export-docx.js               Markdown → Word no padrão visual Sogest
└── .github/
    └── ISSUE_TEMPLATE/{gap.yml, suggestion.yml, config.yml}
```

Sem `frontend/`, sem migrations SQL, sem `.github/workflows` de deploy — não
há ambiente para promover.

### O método: a disciplina de gates do OPS, formalizada em git

O OPS já resolve o mesmo problema que o `superpowers:brainstorming` resolve
para software — não produzir entregável final sem entendimento validado —
só que com nomes próprios. Mapeamento direto:

| OPS (Master Build) | Equivalente no método Git |
|---|---|
| Modo Entrevista + Diagnóstico | `superpowers:brainstorming` — perguntas uma a uma, sem gerar entregável |
| Modo Arquitetura | Apresentação do design em seções, aprovação |
| Gate antes de Produção | O gate que já existe no `brainstorming` — nada de doc final sem aprovação explícita |
| Modo Produção | Spec + entregável final em Markdown, commitado |
| Captura Semanal (contínua; capturar ≠ atualizar) | Comentários acumulando numa Issue "semana N" |
| Consolidação Semanal ("agora pode rodar a atualização") | Fechar a issue da semana com um resumo — só quando pedido explicitamente |

`docs/how-we-conduct-a-deliverable.md` documenta essa tabela como o método
oficial do template.

### Onde cada artefato do OPS mora no novo modelo

- **Master Briefing** → `docs/PROJECT.md` preenchido — documento-mestre de
  contexto/governança, mesmo papel que já cumpre no template de software.
- **Portfólio Único** (Executiva / Resumo / Base Total / Log) → **não gera
  Excel**: vira o **GitHub Project board nativo**. Campos customizados
  (Gate, Pilar, Cluster, Impacto, Owner) + múltiplas *views* salvas cobrem as
  4 abas (view agrupada por Gate = Executiva; view filtrada = Resumo; lista
  completa = Base Total). O Log de Atualizações é o histórico de atividade
  nativo de cada issue — não precisa ser mantido à mão.
- **Memória Discursiva** → `docs/memoria-discursiva.md`.
- **Weekly Result Report** → `templates/weekly-report-template.md`,
  preenchido a cada consolidação.

### Privacidade por padrão

Diferente do template de Software (público, sem segredo nenhum na
estrutura), projetos do tipo Metodologia e Gestão **nascem privados por
padrão** — frequentemente carregam informação sensível de cliente
(governança, riscos, pontos sensíveis, memória discursiva). O onboarding
continua perguntando privado/público; só muda a recomendação padrão exibida.

### Exportação para Word/Excel — camada opcional, não fonte de verdade

`scripts/export-docx.js` generaliza o `gen-spec-docx.js` já existente em
`prisma-cfo` (paleta de cores, tabelas, cabeçalhos no padrão visual Sogest).
Roda sob demanda quando um documento precisa ser entregue formalmente a um
cliente. A fonte de verdade continua sendo o Markdown versionado no git —
o `.docx` é sempre gerado a partir dele, nunca editado à parte.

## Consequências

- Dois repositórios-template em vez de um; a skill de onboarding precisa
  ramificar em "Fase 1" (tipo de projeto) antes de decidir qual template
  usar — pré-voo, labels, board e milestones continuam compartilhados.
- O Portfólio Único deixa de ser um artefato Excel separado — depende do
  GitHub Projects (v2) suportar campos customizados e múltiplas views
  salvas por conta gratuita/plano usado; a ser confirmado na implementação.
- `export-docx.js` depende do pacote `docx` (mesma lib usada em
  `gen-spec-docx.js`) — precisa de Node.js instalado na máquina de quem
  exporta, não só `gh`/git.
- Escopo deste design é a estrutura de condução (tracking + método + docs).
  Não inclui automação de geração de Excel (substituído pelo board nativo)
  nem replicação campo-a-campo do OPS — viram convenção documentada,
  preenchida durante o uso, não geradas automaticamente pelo instalador.
- O "Sistema de Gestão da Sogest" (projeto-mãe) será o primeiro uso real
  deste template, criado depois que a implementação estiver pronta — não
  faz parte deste design.
