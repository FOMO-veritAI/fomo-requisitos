---
description: "Task list for FOP — verificação de alegações por IA"
---

# Tasks: FOP — Verificação de alegações por IA

**Input**: Design documents from `/specs/001-fop-verificacao-alegacoes/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Incluídos — o NFR-012 (plan.md) exige cobertura ≥70% nos módulos críticos (veredito, filtros temporais, autorização, revisão) e suíte ≤5 min.

**Organization**: Tarefas agrupadas por user story para implementação e teste independentes.

## Nota de Handoff (implementação por outro fluxo/modelo)

- Este `tasks.md` será executado por **outro modelo/fluxo de planejamento**, orientado por outro membro do grupo. Trate-o como um **backlog ordenado por dependência** — um mapa de execução —, não como substituto do planejamento detalhado de cada tarefa.
- **As especificações técnicas herdadas do `fluxo-ia-fop-v3.md` devem ser mantidas**: arquitetura do fluxo de IA, bibliotecas, modelos, parâmetros, limiares, janelamento e templates são o desenho definitivo do grupo. O mesmo vale para `plan.md`, `research.md`, `data-model.md` e `contracts/`.
- **Caminhos de arquivo e nomes de módulos são sugestões** coerentes com o `plan.md`; podem ser refinados pelo fluxo implementador **desde que os FRs/NFRs** (`spec.md` e `docs/FOP-Requisitos-FR-NFR.md`) **e os contratos** (`contracts/`) **sejam honrados**.
- Cada tarefa traz, além da ação, o **resultado esperado** — o critério de aceite mínimo daquela tarefa. Tarefas podem ser subdivididas, desde que o resultado seja preservado.
- **Pacote de contexto recomendado**: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `fluxo-ia-fop-v3.md`, `docs/FOP-Requisitos-FR-NFR.md`.
- **Checklist de cobertura/aceitação**: `docs/FOP-Requisitos-Detalhados.md` (132 RF/RNF atômicos) — use-o para verificar, item a item, que nenhum sub-requisito ficou de fora; ele complementa (não substitui) a execução guiada por `spec.md`, `plan.md`, `contracts/` e por este backlog.

## Format: `[ID] [P?] [Story] Ação com caminho → Resultado esperado`

- **[P]**: Pode rodar em paralelo (arquivos diferentes, sem dependências)
- **[Story]**: User story (US1–US4)
- Caminhos de arquivo exatos nas descrições
- **Resultado esperado**: critério de aceite mínimo da tarefa (verificável)

## Path Conventions

- **Single project**: `src/fop/`, `app/`, `tests/` na raiz (conforme plan.md)
- `data/` (gitignored): bases P1/P2 e índices; `config/config.json`: parâmetros do v3

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Inicialização do projeto e estrutura base

- [ ] T001 Criar a estrutura de diretórios do projeto (`src/fop/{api,core,prep/{p1_checks,p2_evidence,p3_calibration},pipeline/{select_claims,attribute,match_checks,retrieve,verdict,calibrate,explain},retrieval,storage,observability,eval,models,security}`, `app/`, `tests/{contract,integration,unit}`, `config/`, `data/`, `notebooks/`) conforme plan.md — **Resultado esperado:** árvore de diretórios criada exatamente como no plan.md, com pacotes Python importáveis
- [ ] T002 Inicializar projeto Python 3.11 com `uv` e dependências em `pyproject.toml` (spaCy, dateparser, trafilatura, datasketch, rank_bm25, sentence-transformers, faiss-cpu, transformers, scikit-learn, fastapi, uvicorn, streamlit, pandas, pydantic, pytest, pytest-cov) — **Resultado esperado:** `uv sync` instala tudo sem erro e os imports principais funcionam
- [ ] T003 [P] Configurar lint/format (ruff) e pytest em `pyproject.toml` (NFR-012) — **Resultado esperado:** `uv run pytest` e `uv run ruff check .` executam na raiz sem erro de configuração
- [ ] T004 [P] Criar `config/config.json` com os parâmetros iniciais do v3 (`max_alegacoes=10`, `top_k_checagens=10`, `idade_max_checagem_meses=12`, `top_k_bm25=50`, `top_k_denso=50`, `top_n_rerank=30`, `top_n_final=5`, `max_palavras_trecho_exibido=25`) e o carregador `src/fop/config.py` — **Resultado esperado:** parâmetros carregados por `fop.config`, com defaults documentados e validação de tipos
- [ ] T005 [P] Configurar variáveis de ambiente em `.env.example` (`GOOGLE_FACTCHECK_API_KEY`, `FOP_LLM_PROVIDER` opcional) e leitura segura em `src/fop/config.py` (NFR-008: segredos fora do código) — **Resultado esperado:** nenhuma credencial no código; chaves lidas apenas de variáveis de ambiente
- [ ] T006 [P] Criar `data/README.md` documentando que as bases P1/P2 e índices **não** são versionados (`data/` já está no `.gitignore`) — **Resultado esperado:** `git status` limpo após rodar os builders; documentação clara do caráter local das bases

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Infraestrutura e preparação P1–P3 que bloqueiam todas as histórias

**⚠️ CRITICAL**: Nenhuma user story pode começar antes desta fase

- [ ] T007 Criar schemas de domínio (pydantic) em `src/fop/models/` — Notícia, Alegação (`tipo_atribuicao` ∈ {fonte_citada, declaracao, nenhuma}, `sensivel_ao_tempo`, `ambigua`), Checagem, Casamento, Trecho (`data_publicacao`), Veredito (`origem` ∈ {agencia, fop}, `motivo` ∈ {agencias_divergem, evidencias_divergem, baixa_confianca, ambigua, numerico_divergente, mudanca_veredito}, `status` ∈ {final, pendente_revisao}), Explicação (`gerado_por` ∈ {template, llm}) e Fila de revisão — conforme data-model.md — **Resultado esperado:** entidades validáveis com enums/obrigatoriedades de data-model.md; valores inválidos são rejeitados
- [ ] T008 [P] Implementar camada SQLite em `src/fop/storage/db.py` (tabelas `checagens`, `trechos`, `alegacoes`, `vereditos`, `fila_revisao`, `auditoria`, `noticias`) e repositórios — **Resultado esperado:** banco criado com as 7 tabelas e CRUD básico testado
- [ ] T009 [P] Implementar log de auditoria e snapshots em `src/fop/storage/audit.py` (`snapshot_p1`, `snapshot_p2`, `versao_config`, modelos, substituições, timestamps) — FR-015 — **Resultado esperado:** todo veredito rastreável a snapshots/versões; registro imutável
- [ ] T010 [P] Implementar retenção transitória do texto em `src/fop/storage/retention.py` (descartar após publicação/conclusão; manter só durante verificação/revisão pendente) — FR-025 — **Resultado esperado:** texto ausente do armazenamento após publicação/conclusão; presente apenas durante verificação/revisão pendente
- [ ] T011 [P] Implementar papéis e autorização em `src/fop/security/roles.py` (publisher verificado, reader, reviewer, admin; `403 publisher_not_verified`) — FR-001/FR-016/FR-018 — **Resultado esperado:** submissão por não verificado retorna 403; reader/reviewer com acessos corretos; SC-007
- [ ] T012 Criar esqueleto FastAPI + `GET /health` em `src/fop/api/main.py` (retornar versões de snapshots) — contracts/http-api.md — **Resultado esperado:** `GET /api/v1/health` responde `{status, pipeline_version, snapshots}`
- [ ] T013 [P] Implementar middleware de erro (envelope `{error:{code,message}}`) e logging JSON por verificação em `src/fop/api/middleware.py` — NFR-006 — **Resultado esperado:** erros no envelope padrão; log JSON por verificação com tempo por etapa e versões
- [ ] T014 [P] Criar esqueleto da UI Streamlit com modos submissão/leitura/revisão em `app/streamlit_app.py` — **Resultado esperado:** app abre e alterna entre os três modos sem erro
- [ ] T015 [P] Implementar serviço de embeddings único (e5-base ou bge-m3 via config) em `src/fop/retrieval/embeddings.py` (vetores normalizados; prefixos `query:`/`passage:` se e5) — **Resultado esperado:** embeddings determinísticos e normalizados; troca de modelo exige reindexação (documentado)
- [ ] T016 [P] Implementar **P1 — base de checagens** em `src/fop/prep/p1_checks/build.py` (Google Fact Check Tools API, Fato ou Boato, FactCenter; mapeamento veredito→4 classes; dedup; grupos divergentes → `agencias_divergem`; índice FAISS + `snapshot_p1`) — **Resultado esperado:** base deduplicada com classes mapeadas e índice FAISS; divergência entre agências marcada como `conflitante`/`agencias_divergem`
- [ ] T017 [P] Implementar **P2 — base de evidência** em `src/fop/prep/p2_evidence/build.py` (fontes oficiais; extração com trafilatura; MinHash; trechos ~150/30 palavras; índices BM25 + vetorial; `snapshot_p2`) — **Resultado esperado:** trechos oficiais com fonte/domínio/data, indexados em BM25+vetorial, com `snapshot_p2`
- [ ] T018 [P] Implementar **P3 — limiares e calibração** em `src/fop/prep/p3_calibration/run.py` (split train-A/train-B do AVERITEC-BR; `similaridade_minima`, `limiar_equivalencia`, `limiar_suporta`, `limiar_refuta`, `limiar_abstencao`; Platt por classe; seeds fixas; saída `versao_config`) — **Resultado esperado:** `config.json` com limiares escolhidos no train-A e calibradores no train-B, seeds registradas
- [ ] T019 [P] Testes unitários das fundações em `tests/unit/test_models.py`, `tests/unit/test_storage.py`, `tests/unit/test_roles.py` — **Resultado esperado:** suite de fundação passando e cobrindo validações críticas

**Checkpoint**: Fundação pronta — user stories podem começar em paralelo

---

## Phase 3: User Story 1 - Publicador verificado submete e recebe a verificação antes de publicar (Priority: P1) 🎯 MVP

**Goal**: Submissão/ressubmissão pelo publicador com resultado por alegação (4 classes, evidências, explicação) e bloqueio de publicação em casos sensíveis

**Independent Test**: submeter notícia como publicador verificado e receber classes + evidências + explicação; notícia com caso sensível não publica até revisão

### Tests for User Story 1

- [ ] T020 [P] [US1] Teste de contrato para `POST /verifications` e `POST /verifications/{id}/versions` (202, 403, 409, 422 — incluindo `unsupported_language`) em `tests/contract/test_verifications.py` — **Resultado esperado:** contratos cobertos pelos testes, que falham antes da implementação
- [ ] T021 [P] [US1] Teste de integração do fluxo submissão → resultado por alegação em `tests/integration/test_submission_flow.py` — **Resultado esperado:** fluxo ponta a ponta retorna 4 classes com evidências e explicação
- [ ] T022 [P] [US1] Testes unitários dos passos 1–3 (seleção, atribuição, casamento) em `tests/unit/test_select_claims.py`, `tests/unit/test_attribute.py`, `tests/unit/test_match_checks.py` — **Resultado esperado:** limite de 10 alegações, atribuição e casamento bidirecional cobertos por casos de teste
- [ ] T023 [P] [US1] Testes unitários dos passos 4–5 (recuperação, veredito NLI+regras) em `tests/unit/test_retrieve.py`, `tests/unit/test_verdict.py` — **Resultado esperado:** filtros temporais, agregação das 4 classes e ausência→inconclusiva cobertos

### Implementation for User Story 1

- [ ] T024 [US1] Validar idioma da submissão (pt-BR) em `src/fop/api/verifications.py` e `src/fop/pipeline/select_claims/language.py` (detecção; rejeitar não-pt-BR com 422 `unsupported_language`) — FR-014 — **Resultado esperado:** submissões em outro idioma retornam 422; notícias em pt-BR prosseguem
- [ ] T025 [P] [US1] Passo 1 — seleção de frases verificáveis em `src/fop/pipeline/select_claims/select.py` (regras: número, data, NER, verbo de declaração; `max_alegacoes=10`; registrar frases excluídas) — **Resultado esperado:** até 10 alegações por notícia, com contagem de frases excluídas disponível ao publicador
- [ ] T026 [P] [US1] Passo 2 — descontextualização e atribuição em `src/fop/pipeline/attribute/attribute.py` (NER spaCy; substituições só com candidato único, senão `ambigua=true`; `tipo_atribuicao`; datas relativas; `sensivel_ao_tempo`) — **Resultado esperado:** alegações autocontidas com `tipo_atribuicao`, `atribuida_a`/`fonte_citada`, `data_referencia`, `sensivel_ao_tempo` e `ambigua`
- [ ] T027 [US1] Passo 3 — casamento com checagens em `src/fop/pipeline/match_checks/match.py` (top-k por embedding; NLI bidirecional ≥ `limiar_equivalencia`; filtro `idade_max_checagem_meses`; divergência → `agencias_divergem`) — **Resultado esperado:** herda veredito de agência apenas com equivalência bidirecional e dentro da idade máxima; divergência vira `conflitante`
- [ ] T028 [US1] Cache de alegações repetidas do passo 3 invalidado por snapshot em `src/fop/storage/cache.py` (chave: alegação normalizada + snapshots) — v3 Operação/NFR-014 — **Resultado esperado:** resultado reaproveitado para alegação idêntica com os mesmos snapshots; invalidado quando os snapshots mudam
- [ ] T029 [P] [US1] Passo 4 — recuperação em `src/fop/pipeline/retrieve/retrieve.py` (BM25 50 + densa 50 + RRF; remover trechos da própria notícia; filtros temporais; `top_n_final=5`) — **Resultado esperado:** até 5 trechos por alegação, nunca da própria notícia, respeitando o corte temporal
- [ ] T030 [P] [US1] Passo 5 — veredito em `src/fop/pipeline/verdict/verdict.py` (janelas de 1–2 frases; NLI; `limiar_suporta`/`limiar_refuta`; agregação por regras; inconclusiva sem confiança) — **Resultado esperado:** exatamente uma das 4 classes, com motivo; inconclusiva nunca recebe confiança
- [ ] T031 [US1] Passo 6 — calibração e abstenção em `src/fop/pipeline/calibrate/calibrate.py` (confiança por classe; abaixo de `limiar_abstencao` → inconclusiva `baixa_confianca`) — **Resultado esperado:** confiança calibrada por classe e abstenção aplicada pelo limiar
- [ ] T032 [US1] Passo 6 — roteamento de revisão em `src/fop/pipeline/calibrate/review_router.py` (refutada; conflitante com pessoa; `ambigua`; conflito numérico; mudança de veredito — FR-020/FR-023) — **Resultado esperado:** 100% dos casos sensíveis entram na fila de revisão (SC-008)
- [ ] T033 [US1] Passo 7 — explicação por templates em `src/fop/pipeline/explain/explain.py` (por origem e `tipo_atribuicao`; janela decisiva ≤ `max_palavras_trecho_exibido`; "Declaração de [X]"; "A notícia atribui este dado a [fonte]") — FR-011 — **Resultado esperado:** explicação neutra com citações, identificando a quem o resultado se refere
- [ ] T034 [US1] Orquestrador do pipeline em `src/fop/core/pipeline.py` (falhas → inconclusiva/template; nunca refutada por ausência) — FR-008, NFR-005 — **Resultado esperado:** falha de componente não derruba a verificação; ausência de evidência vira inconclusiva
- [ ] T035 [US1] Endpoint `POST /api/v1/verifications` (publicador; 202 assíncrono; 403/422) em `src/fop/api/verifications.py` — **Resultado esperado:** submissão aceita com 202 + id; 403/422 conforme contrato
- [ ] T036 [US1] Endpoint `POST /api/v1/verifications/{noticia_id}/versions` (até 3 versões; histórico ligado; mudança de veredito → revisão) em `src/fop/api/verifications.py` — **Resultado esperado:** até 3 versões com histórico; mudança refutada→suportada/inconclusiva entra em revisão
- [ ] T037 [US1] Portão de publicação em `src/fop/storage/publication.py` (pendente de revisão → `publication_allowed=false`; publicador consulta o status no resultado) — FR-021 — **Resultado esperado:** notícia com pendência não publica; status consultável pelo publicador (notificação passiva)
- [ ] T038 [US1] UI de submissão (publicador) em `app/streamlit_app.py` — **Resultado esperado:** publicador cola a notícia, envia, acompanha status/resultado, vê as frases excluídas e pode ressubmeter (até 3 versões)
- [ ] T039 [US1] Registrar auditoria por verificação em `src/fop/core/pipeline.py` (snapshots, versões, substituições, resultado) — FR-015 — **Resultado esperado:** toda verificação reproduzível e auditável

**Checkpoint**: User Story 1 funcional e testável isoladamente (MVP)

---

## Phase 4: User Story 2 - Leitor vê notícias publicadas com classificação e evidências, sem bloqueio (Priority: P2)

**Goal**: Leitura das notícias publicadas com classificação/evidências em layout simétrico; refutadas visíveis

**Independent Test**: acessar como usuário comum uma notícia publicada refutada e ver classificação + argumentos lado a lado, sem ação de submissão

### Tests for User Story 2

- [ ] T040 [P] [US2] Teste de contrato de `GET /verifications/{id}` (processing/pendente/verificada, `publication_allowed`) em `tests/contract/test_reader_api.py` — **Resultado esperado:** três estados e `publication_allowed` validados por teste
- [ ] T041 [P] [US2] Teste de integração de leitura de refutada publicada em `tests/integration/test_reader_flow.py` — **Resultado esperado:** refutada publicada acessível com classificação e argumentos

### Implementation for User Story 2

- [ ] T042 [US2] Endpoint `GET /api/v1/verifications/{noticia_id}` em `src/fop/api/verifications.py` (leitura por qualquer usuário; FR-017) — **Resultado esperado:** leitura anônima retorna resultado completo quando publicado
- [ ] T043 [US2] Serialização do resultado com `policy.blocked=false` e `policy.full_text_retention="transient"` em `src/fop/api/schemas.py` — **Resultado esperado:** campos de política presentes em 100% das respostas
- [ ] T044 [US2] Visibilidade apenas de publicadas e garantia de não bloqueio/ocultação em `src/fop/storage/publication.py` — FR-010 — **Resultado esperado:** nenhuma notícia ocultada por classe; apenas publicadas visíveis ao leitor
- [ ] T045 [US2] UI de leitura com layout simétrico (favorável × contrária) em `app/streamlit_app.py` — FR-019 — **Resultado esperado:** colunas simétricas por alegação, sem ação de submissão

**Checkpoint**: US1 e US2 funcionam independentemente

---

## Phase 5: User Story 3 - Conferir fontes, atribuição e incerteza (Priority: P3)

**Goal**: Transparência de fontes/datas/janela decisiva, atribuição e incerteza

**Independent Test**: abrir o detalhe de uma classificação e ver fonte/data/janela e a atribuição; casos sem evidência exibem incerteza

### Tests for User Story 3

- [ ] T046 [P] [US3] Teste de schema/contrato dos campos de evidência e atribuição (`evidence[].stance/decisive`, `tipo_atribuicao`, `atribuida_a`, `fonte_citada`) em `tests/contract/test_result_schema.py` — **Resultado esperado:** schema rejeita resultados sem fonte/data/decisive ou sem atribuição

### Implementation for User Story 3

- [ ] T047 [US3] Expor evidências detalhadas (fonte, data, janela decisiva) no resultado em `src/fop/api/verifications.py` e `src/fop/api/schemas.py` — FR-009 — **Resultado esperado:** 100% das classificações com ao menos uma evidência com fonte/data (SC-002)
- [ ] T048 [US3] Exibir atribuição ("Declaração de [X]", "A notícia atribui este dado a [fonte]") em `src/fop/pipeline/explain/explain.py` e na UI — **Resultado esperado:** texto de atribuição correto por `tipo_atribuicao`
- [ ] T049 [US3] Exibir incerteza/conflito com motivo e sem confiança na inconclusiva em `app/streamlit_app.py` e `src/fop/api/schemas.py` — FR-013 — **Resultado esperado:** incerteza/conflito explícitos com motivo; inconclusiva sem confiança

**Checkpoint**: US1–US3 funcionam independentemente

---

## Phase 6: User Story 4 - Revisor humano decide os casos pendentes (Priority: P3)

**Goal**: Fila de revisão com evidências e diff entre versões; decisão registrada

**Premissa do MVP**: um **revisor designado (papel único)**; quem revisa (moderação, veículo ou ambos) é decisão da administração (v3, item 1).

**Independent Test**: gerar caso refutado → ver na fila com evidências e diff → decidir → publicação liberada

### Tests for User Story 4

- [ ] T050 [P] [US4] Teste de contrato dos endpoints da fila de revisão em `tests/contract/test_review_api.py` — **Resultado esperado:** apenas revisor acessa; itens com motivo/evidências/diff
- [ ] T051 [P] [US4] Teste de integração caso sensível → revisão → liberação em `tests/integration/test_review_flow.py` — **Resultado esperado:** fluxo completo revisão→publicação validado

### Implementation for User Story 4

- [ ] T052 [US4] Endpoint `GET /api/v1/review-queue` (somente revisor) em `src/fop/api/review.py` — FR-020 — **Resultado esperado:** fila listada por status com todos os campos do item
- [ ] T053 [US4] Endpoint `POST /api/v1/review-queue/{item_id}/decision` (aceitar/rejeitar/ajustar + justificativa) em `src/fop/api/review.py` — **Resultado esperado:** decisão registrada com revisor, timestamp e justificativa
- [ ] T054 [US4] Diferença entre versões do texto para o revisor em `src/fop/storage/audit.py` (C4) — **Resultado esperado:** revisor recebe o diff entre versões da mesma notícia
- [ ] T055 [US4] UI da fila de revisão com evidências e diff em `app/streamlit_app.py` — **Resultado esperado:** revisor decide pela interface com o contexto completo
- [ ] T056 [US4] Métricas da fila (tamanho, tempo de espera) em `src/fop/observability/metrics.py` — NFR-006 — **Resultado esperado:** tamanho e tempo de espera da fila observáveis

**Checkpoint**: Todas as user stories independentemente funcionais

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Qualidade, avaliação, entrega e conformidade

- [ ] T057 [P] Acessibilidade WCAG 2.1 AA nas telas principais (contraste ≥4,5:1, teclado, rótulos/ARIA) em `app/streamlit_app.py` — NFR-010 — **Resultado esperado:** telas principais em conformidade AA verificada
- [ ] T058 [P] Métricas de observabilidade (p50/p95, taxa de abstenção, fila) em `src/fop/observability/metrics.py` — NFR-006 — **Resultado esperado:** métricas agregadas disponíveis e registradas
- [ ] T059 [P] Benchmark de carga E9 (10 alegações; CPU; pico de 10×; custo) em `notebooks/benchmark_e9.py` — NFR-001/002/003 — **Resultado esperado:** tempo/custo por notícia e comportamento no pico medidos e documentados
- [ ] T060 Implementar avaliação offline em `src/fop/eval/run.py` (macro-F1 por classe, refutada × inconclusiva, calibração, abstenção, métricas com/sem passo 3, bootstrap) — **Resultado esperado:** relatório de avaliação com ICs, separando refutada de inconclusiva
- [ ] T061 [P] Verificações anti-vazamento em `src/fop/eval/leakage.py` (dedup do teste contra P1; remover domínios de agências e documentos posteriores) — research.md §13 — **Resultado esperado:** teste deduplicado contra P1; sem agências/documentos futuros
- [ ] T062 [P] Hardening de segurança: garantir nenhuma chamada externa com texto não publicado; segredos em env; HTTPS em `src/fop/security/` — NFR-008/FR-024 — **Resultado esperado:** 0% do texto não publicado enviado a terceiros no MVP (SC-009)
- [ ] T063 [P] Checklist de licenças e conformidade (AVeriTeC CC BY-NC, fontes oficiais, LGPD/parecer jurídico) em `docs/licensing.md` — NFR-009 — **Resultado esperado:** checklist jurídico documentado como pré-requisito do piloto
- [ ] T064 [P] Validar os cenários do `quickstart.md` (1–9) e registrar resultados em `docs/validation.md` — **Resultado esperado:** 9 cenários executados e documentados
- [ ] T065 [P] Atualizar `README.md` com comandos reais de preparação/execução após a implementação — **Resultado esperado:** README reproduzível do zero
- [ ] T066 [P] Configurar deploy da demo em CPU (Hugging Face Spaces ou contêiner) com smoke test em `docs/deploy.md` — NFR-004/NFR-013 — **Resultado esperado:** demo acessível com `/health` respondendo e disponibilidade ≥95% na janela de avaliação
- [ ] T067 [P] Adicionar medição de cobertura e gate em CI (`pytest --cov`, mínimo 70% nos módulos críticos) em `.github/workflows/ci.yml` — NFR-012 — **Resultado esperado:** pipeline falha se a cobertura dos módulos críticos ficar abaixo de 70%

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: sem dependências
- **Foundational (Phase 2)**: depende do Setup — **BLOQUEIA** todas as user stories (inclui P1/P2/P3)
- **User Stories (Phases 3–6)**: dependem da Fundação
  - US1 (P1) → MVP
  - US2 (P2) depende de resultados armazenados por US1 (mas testável com fixtures)
  - US3 (P3) depende de US1/US2 para exibição, testável com fixtures
  - US4 (P3) depende do roteamento de revisão (US1)
- **Polish (Phase 7)**: depende das histórias desejadas

### Within Each User Story

- Testes antes da implementação quando incluídos
- Modelos → serviços → endpoints → UI
- Núcleo antes da integração

### Parallel Opportunities

- Setup: T003–T006 em paralelo
- Fundação: T008–T019 (exceto T012/T018 quando dependem de config) em paralelo; T016/T017 podem rodar juntos
- US1: testes T020–T023 em paralelo; implementação T024–T026 e T029–T030 em paralelo (arquivos distintos); T027→T028; T031–T039 em sequência
- US2: T040–T041 em paralelo; T042–T045 em sequência
- US3: T046 e T047–T049
- US4: T050–T051 em paralelo; T052–T056 em sequência
- Polish: T057–T059, T061–T067 em paralelo; T060 após T059

---

## Parallel Example: User Story 1

```bash
# Testes em paralelo:
Task: "Teste de contrato POST /verifications em tests/contract/test_verifications.py"
Task: "Testes unitários dos passos 1-3 em tests/unit/test_select_claims.py, test_attribute.py, test_match_checks.py"

# Implementação paralela (arquivos distintos):
Task: "Passo 1 — seleção em src/fop/pipeline/select_claims/select.py"
Task: "Passo 2 — atribuição em src/fop/pipeline/attribute/attribute.py"
Task: "Passo 4 — recuperação em src/fop/pipeline/retrieve/retrieve.py"
Task: "Passo 5 — veredito em src/fop/pipeline/verdict/verdict.py"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Phase 1: Setup
2. Phase 2: Foundational (P1/P2/P3 + infra) — **crítico**
3. Phase 3: US1
4. **PARAR e VALIDAR** a US1 isoladamente (submissão → resultado)
5. Demo/deploy se pronta

### Incremental Delivery

1. Fundação pronta
2. US1 → validar → demo (MVP)
3. US2 → validar → demo
4. US3 → validar → demo
5. US4 → validar → demo

---

## Notes

- `[P]` = arquivos diferentes, sem dependências
- Cada história é completável e testável isoladamente
- Testes falham antes da implementação (quando incluídos)
- Commit após cada tarefa ou grupo lógico
- Evitar: tarefas vagas, conflito no mesmo arquivo, dependências entre histórias
- Referências cruzadas: FR/NFR consolidados em `docs/FOP-Requisitos-FR-NFR.md`; desenho em `fluxo-ia-fop-v3.md`
- As especificações técnicas herdadas do v3 **devem ser mantidas**; ajustes de detalhe são permitidos desde que FRs/NFRs e contratos sejam preservados (ver Nota de Handoff)
