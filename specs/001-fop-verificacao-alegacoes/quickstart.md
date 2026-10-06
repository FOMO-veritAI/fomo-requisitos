# Quickstart: FOP — Verificação de alegações por IA

**Feature**: `001-fop-verificacao-alegacoes` | **Date**: 2026-09-27 | **Plan**: [plan.md](./plan.md) | **Fonte do desenho**: `fluxo-ia-fop-v3.md`

Guia de validação ponta a ponta do MVP. Sem implementação — apenas pré-requisitos, comandos e resultados esperados.

## Pré-requisitos

- Python 3.11 e `uv` (ou `venv` + `pip`)
- Git
- **Nenhuma API de LLM é necessária** no MVP (explicação por templates; veredito por NLI+regras)
- Opcional (incremento): `Ollama` com modelo 7–8B para explicação local
- Opcional (preparação): GPU para ajustes/calibração — a execução roda em CPU
- Chave da Google Fact Check Tools API (preparação P1) e espaço para as bases P1/P2 em `data/`

## Preparação (P1–P3)

```bash
uv sync

# P1 — base de checagens das agências (4 classes + índice)
uv run python -m fop.prep.p1_checks.build --out data/p1

# P2 — base de evidência em fontes oficiais (trechos + BM25 + vetorial)
uv run python -m fop.prep.p2_evidence.build --out data/p2

# P3 — limiares (train-A) e calibradores (train-B) do AVERITEC-BR
uv run python -m fop.prep.p3_calibration.run --config config/config.json
```

## Executar

```bash
# API
uv run uvicorn fop.api.main:app --port 8000

# UI (em outro terminal)
uv run streamlit run app/streamlit_app.py
```

## Cenários de validação

### 1. Publicador verificado submete e recebe o resultado (US1 / FR-001–FR-009)

1. Na UI, modo **submissão** (publicador `jornal-exemplo`, pré-aprovado).
2. Colar notícia ≤1.000 palavras; enviar.
3. **Esperado** (≤2 min): por alegação, uma das 4 classes, evidências com fonte/data, trecho decisivo e explicação neutra.
4. Via API:
   ```bash
   curl -s -X POST localhost:8000/api/v1/verifications \
     -H "X-Publisher-Id: jornal-exemplo" -H "Content-Type: application/json" \
     -d '{"text":"...","title":"...","language":"pt-BR"}'
   curl -s localhost:8000/api/v1/verifications/<noticia_id>
   ```

### 2. Ausência de evidência → "inconclusiva" (FR-008)

- Submeter notícia sobre tema sem evidência na base oficial.
- **Esperado**: `not_enough_evidence` (nunca `refuted`) e aviso de incerteza.

### 3. Evidências divergentes → "conflitante" (FR-013)

- Submeter alegação com fontes confiáveis em direções opostas (casos de teste curados).
- **Esperado**: `conflicting_evidence`, com `motivo` (`evidencias_divergem` ou `agencias_divergem`) e trechos lado a lado.

### 4. Caso sensível → fila de revisão e bloqueio de publicação (US4 / FR-020/FR-021)

1. Submeter notícia que gere **refutada** (ou conflitante com pessoa, ambígua, etc.).
2. **Esperado**: `status = pendente_revisao`, `publication_allowed = false`.
3. Como revisor: listar a fila e decidir.
   ```bash
   curl -s localhost:8000/api/v1/review-queue?status=pendente -H "X-Reviewer-Id: revisor-1"
   curl -s -X POST localhost:8000/api/v1/review-queue/<item_id>/decision \
     -H "X-Reviewer-Id: revisor-1" -H "Content-Type: application/json" \
     -d '{"decisao":"aceitar","justificativa":"..."}'
   ```
4. **Esperado**: após a decisão, a notícia pode ser publicada.

### 5. Leitor vê refutada com layout simétrico e atribuição (US2/US3 / FR-010/FR-011/FR-019)

- Abrir no modo **leitura** uma notícia publicada refutada.
- **Esperado**: classificação, evidências favoráveis × contrárias lado a lado, `policy.blocked = false`; alegações do tipo declaração exibem "Declaração de [X]:" e as com dado atribuído exibem "A notícia atribui este dado a [fonte]:".

### 6. Papéis (FR-016/FR-018 / SC-007)

- Submeter com `X-Publisher-Id` desconhecido/revogado → `403 publisher_not_verified`.
- No modo leitura não existe ação de submissão.

### 7. Correção e ressubmissão (FR-022/FR-023 / SC-010)

- Ressubmeter a mesma notícia corrigida (até 3 versões).
- **Esperado**: nova `versao_texto` ligada ao histórico; mudança de refutada → suportada/inconclusiva entra na fila de revisão.

### 8. Confidencialidade e retenção (FR-024/FR-025 / SC-009)

- Monitorar chamadas externas durante uma verificação.
- **Esperado**: nenhuma saída com o texto não publicado; após publicação/conclusão, o texto não consta do armazenamento (só o registro de auditoria).

### 9. Corte temporal (FR-012)

- Submeter notícia cuja única evidência teria data posterior à submissão (e checagem antiga para alegação sensível ao tempo).
- **Esperado**: evidência/checagem não usada; resultado tende a `not_enough_evidence`.

## Testes e avaliação

```bash
# testes unitários/contrato/integração
uv run pytest

# avaliação offline (anti-vazamento: dedup do teste contra P1, remoção de agências e docs posteriores)
uv run python -m fop.eval.run --dataset averitec_br --split test --with-and-without-matching
```

**Metas**: SC-003 (refutada × inconclusiva separadas, desempenho ≥ baselines); SC-005 (≤2 min p95, provisório); SC-007 (100% não verificados negados); SC-008 (100% casos sensíveis revisados antes de publicar); SC-009 (0% do texto para terceiros).

## Observações

- Demo pública em CPU (Hugging Face Spaces); sem API paga obrigatória.
- A precisão de refutada medida na avaliação **não estima** a produção (validade externa) — reajuste no piloto (E11) com o conjunto de produção e parecer jurídico.
