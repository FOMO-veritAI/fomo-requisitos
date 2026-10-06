# Implementation Plan: FOP — Verificação de alegações por IA

**Branch**: `001-fop-verificacao-alegacoes` | **Date**: 2026-09-27 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-fop-verificacao-alegacoes/spec.md` e `fluxo-ia-fop-v3.md` (plano definitivo do grupo).

**Note**: This template is filled in by the `/speckit.plan` command; its definition describes the execution workflow.

## Summary

Construir o MVP da FOP conforme o `fluxo-ia-fop-v3.md`. **Preparação**: P1 (base de checagens de agências → 4 classes + índice), P2 (base de evidência em fontes oficiais → trechos + índices BM25/vetorial), P3 (limiares e calibração no AVERITEC-BR: train-A/train-B). **Execução por notícia**: (1) seleção de frases verificáveis por regras; (2) descontextualização e tipo de atribuição; (3) casamento com checagens existentes (embedding + NLI bidirecional); (4) recuperação híbrida de evidência; (5) veredito por **NLI em janelas + regras** (sem LLM); (6) calibração, abstenção e **fila de revisão humana**; (7) explicação por **templates**. Resultado por alegação entregue ao publicador antes da publicação; refutadas permanecem visíveis ao leitor; texto da notícia com retenção transitória.

## Technical Context

**Language/Version**: Python 3.11

**Primary Dependencies**: `spaCy` (`pt_core_news_lg`) + `dateparser`; `trafilatura`; `datasketch`; `rank_bm25`; `sentence-transformers` + FAISS; `transformers` (NLI `MoritzLaurer/mDeBERTa-v3-base-xnli-multilingual-nli-2mil7`); `scikit-learn` (calibração); FastAPI + Uvicorn; Streamlit; pandas. Incremento: `Ollama`/`llama.cpp` (explicação com LLM local) e `BAAI/bge-reranker-v2-m3`.

**Storage**: SQLite (checagens, trechos, alegações, vereditos, fila de revisão, log de auditoria) + índice FAISS + documentos/cortes em disco. Texto da notícia **transitório** (FR-025).

**Testing**: pytest (`pytest-cov`) para unitários/contrato/integração; avaliação offline com `scikit-learn` (F1 por classe, precisão de refutada, calibração, abstenção)

**Target Platform**: Linux/macOS/Windows para desenvolvimento; demo em Hugging Face Spaces (CPU); ajuste/calibração usa GPU quando disponível — ver `research.md`

**Project Type**: single project (biblioteca Python + API + UI mínima)

**Performance Goals**: notícia típica (≤1.000 palavras, ≤10 alegações) em ≤2 min (p95, provisório); orçamento por etapa: seleção ≤5 s; casamento ≤5 s; recuperação ≤20 s; veredito/NLI ≤60 s; calibração + explicação ≤15 s; ~50 notícias/dia e pico de 10×

**Constraints**: PT-BR; apenas evidência/checagem com data ≤ submissão; base de evidência em **fontes oficiais** no MVP; **texto não publicado não sai para serviços externos**; **veredito não é decidido por LLM**; execução sem GPU; uso acadêmico não comercial; fila de revisão humana; snapshots (`snapshot_p1`, `snapshot_p2`, `versao_config`) por veredito

**Scale/Scope**: MVP acadêmico — 1 interface web, ~4,5 mil exemplos para ajuste/limiares, dezenas de notícias/dia, 4 classes, 3 bases (P1/P2/P3), usuários de demonstração

## Non-Functional Requirements (NFR)

> Consolidado de plan.md (Performance Goals/Constraints/Scale), da spec (SC-005–SC-010, FR-015/FR-020/FR-021/FR-024/FR-025) e do `fluxo-ia-fop-v3.md`. Valores são **metas genéricas de MVP acadêmico**, revisáveis no piloto.

| ID | Categoria | Requisito | Meta / verificação (MVP) |
|----|-----------|-----------|--------------------------|
| NFR-001 | Desempenho | Verificação assíncrona de notícia típica (≤1.000 palavras, ≤10 alegações) | p95 ≤120 s e p50 ≤60 s em CPU; publicador notificado ao concluir (SC-005) |
| NFR-002 | Desempenho | Orçamento por etapa (diagnóstico) | seleção ≤5 s; casamento ≤5 s; recuperação ≤20 s; veredito/NLI ≤60 s; calibração + explicação ≤15 s |
| NFR-003 | Escalabilidade | Concorrência e volume no MVP | 10 submissões simultâneas e ~50/dia sem degradar o p95 em >20%; fila assíncrona absorve pico de 10× |
| NFR-004 | Disponibilidade | Demo/avaliação | ≥95% na janela de avaliação; reinício automático; estado das verificações persistido localmente |
| NFR-005 | Confiabilidade | Falha de componente não é fatal | falha de recuperação/NLI/explicação → inconclusiva ou template; nenhum registro perdido; sem 5xx por dependência opcional |
| NFR-006 | Observabilidade | Log estruturado por verificação (JSON) | tempo por etapa, classe, abstenção, origem e versões (snapshots/config/modelos); métricas p50/p95, abstenção, fila de revisão; retenção de logs 90 dias |
| NFR-007 | Auditabilidade | Rastreabilidade e reprodutibilidade | veredito ligado a snapshots/config; mesmo snapshot + seed → mesmo resultado; registro imutável (FR-015) |
| NFR-008 | Segurança / Confidencialidade | Texto não publicado não sai para serviços externos no MVP | nenhuma chamada externa com o texto (FR-024); segredos em variável de ambiente; acesso por papel; HTTPS |
| NFR-009 | Privacidade / LGPD | Minimização e retenção | texto transitório descartado após publicação/conclusão (FR-025); logs sem conteúdo integral; exclusão sob solicitação; parecer jurídico antes do piloto |
| NFR-010 | Acessibilidade | Telas principais (submissão, revisão e leitura) | WCAG 2.1 AA: contraste ≥4,5:1, teclado, rótulos/ARIA; conteúdo em pt-BR |
| NFR-011 | Usabilidade / Transparência | Leitura compreensível e neutra | classificação e incerteza com fonte/data/janela; layout simétrico; atribuição explícita; leitura típica ≤3 min; nada bloqueado ao leitor (FR-010/FR-019) |
| NFR-012 | Manutenibilidade | Qualidade de código e testes | cobertura ≥70% nos módulos críticos (veredito, filtros temporais, autorização, revisão); suíte ≤5 min em CPU; dependências fixadas; lint |
| NFR-013 | Portabilidade | Ambientes suportados | Linux/macOS/Windows + contêiner; execução **sem GPU**; demo em CPU |
| NFR-014 | Custo operacional | Orçamento do MVP | demo a custo ~zero (CPU + bibliotecas abertas); nenhuma API paga obrigatória; cache de alegações repetidas |

**Notas**: NFR-001/002 são provisórios até o benchmark de carga (E9) e o reajuste do piloto (E11). NFR-008 é atendido por não usar LLM por API no MVP (explicação por template; LLM local apenas como incremento).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- `.specify/memory/constitution.md` ainda é o template não ratificado: não há princípios/gates formais. `Complexity Tracking` não aplicável.
- **Gates provisórios derivados da spec/v3** (verificados no desenho): (1) ausência de evidência nunca vira falsidade (FR-008/FR-013); (2) texto não publicado não sai para serviços externos e tem retenção transitória (FR-024/FR-025); (3) submissão restrita a publicador verificado (FR-001/FR-016); (4) casos sensíveis passam por revisão humana antes de publicar (FR-020/FR-021). Todos **atendidos**.
- **Re-check pós-Fase 1**: mantido — dados e contratos preservam os quatro gates.

## Project Structure

### Documentation (this feature)

```text
specs/001-fop-verificacao-alegacoes/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output
│   ├── http-api.md
│   └── verification-result.schema.json
├── checklists/requirements.md
└── spec.md
```

### Source Code (repository root)

```text
src/fop/
├── api/            # FastAPI: submissão (publisher), leitura (reader), revisão (reviewer)
├── core/           # orquestração do pipeline (passos 1–7) e auditoria
├── prep/
│   ├── p1_checks/      # base de checagens das agências + mapeamento + índice
│   ├── p2_evidence/    # base de evidência (fontes oficiais) + cortes + índices
│   └── p3_calibration/ # limiares (train-A) e calibradores (train-B)
├── pipeline/
│   ├── select_claims/  # passo 1: frases verificáveis (regras + spaCy) + validação de idioma
│   ├── attribute/      # passo 2: descontextualização, atribuição, tempo
│   ├── match_checks/   # passo 3: casamento com checagens (embedding + NLI)
│   ├── retrieve/       # passo 4: BM25 + denso + RRF + filtros
│   ├── verdict/        # passo 5: NLI em janelas + regras
│   ├── calibrate/      # passo 6: confiança, abstenção, fila de revisão
│   └── explain/        # passo 7: templates (LLM local = incremento)
├── retrieval/      # serviço de embeddings compartilhado (P1/P2/consulta)
├── storage/        # SQLite, snapshots, log de auditoria, retenção transitória, cache
├── observability/  # métricas (p50/p95, abstenção, fila)
├── eval/           # avaliação offline e verificações anti-vazamento
├── models/         # schemas pydantic / entidades
└── security/       # papéis publisher/reader/reviewer/admin

app/
└── streamlit_app.py  # UI: submissão, leitura e fila de revisão

notebooks/            # EDA, ablações e experimentos (E1–E11)
data/                 # (gitignored) bases P1/P2, índices e documentos
tests/
├── contract/
├── integration/
└── unit/
```

**Structure Decision**: projeto único Python espelhando as fases do `fluxo-ia-fop-v3.md` (preparação P1–P3, execução passos 1–7). Mantém explícita a regra de ouro: **o veredito é NLI+regras**; o LLM aparece apenas no incremento de explicação.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

Não aplicável: Constitution Check sem violações.
