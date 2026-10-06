# fomo-requisitos — Requisitos da plataforma FOMO e da IA veritAI

**Organização**: FOMO-veritAI
**Origem**: cópia curada dos artefatos de requisitos do repositório de trabalho do Grupo 5 (Residência em IA e Análise de Dados — Instituto Eldorado + UnB/FGA). Foram **excluídos** ferramentas de sincronização de sessões, scripts locais e configurações de máquina.

**Terminologia**: **FOMO** = plataforma; **veritAI** = IA de verificação de alegações (documentada originalmente como "FOP — Fact On Publication").

---

## Estrutura

- **`docs/`** — documentos de requisitos:
  - `FOP-Requisitos-Sistema-Usuario.md` — requisitos de usuário (US1–US4) e de sistema (FR/NFR) consolidados.
  - `FOP-Requisitos-Detalhados.md` — 132 requisitos atômicos (106 RF + 26 RNF); checklist de aceitação.
  - `FOP-Requisitos-FR-NFR.md` — consolidado com trechos de origem.
  - `FOP-Matriz-Rastreabilidade.md` — US ↔ FR/NFR ↔ SC ↔ tarefas ↔ interfaces.
  - `FOP-Interfaces-Sistema.md` — contratos HTTP e contrato de dados.
- **`specs/001-fop-verificacao-alegacoes/`** — especificação no formato Spec Kit: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`, `tasks.md`, `checklists/`.
- **`.specify/memory/constitution.md`** — princípios do projeto (v1.0.0).
- **`.specify/assessments/takta/`** — funil de discovery (intake, research, problem, concept e decisão).
- **`fluxo-ia-fop-v3.md`** — desenho definitivo do fluxo de IA (veritAI).
- **`esboco.txt`** — esboço original do projeto.

## Ordem de leitura sugerida

1. `esboco.txt` e `.specify/assessments/takta/decision.md` — origem e decisão.
2. `specs/001-fop-verificacao-alegacoes/spec.md` e `fluxo-ia-fop-v3.md` — o que o sistema faz.
3. `docs/FOP-Requisitos-Sistema-Usuario.md` e `docs/FOP-Requisitos-Detalhados.md` — requisitos de usuário e de sistema.
4. `specs/001-fop-verificacao-alegacoes/plan.md`, `data-model.md` e `contracts/` — decisões técnicas e interfaces.
5. `docs/FOP-Matriz-Rastreabilidade.md` — rastreabilidade ponta a ponta.

## Estado

- Especificação em **Draft**, pronta para implementação.
- A implementação segue nos repositórios `fomo` (plataforma) e `veritai` (IA).
- Os nomes "FOP" nos documentos referem-se à **veritAI**; a renomeação terminológica pode ser feita em uma revisão futura.
