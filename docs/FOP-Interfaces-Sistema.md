# FOP — Interfaces de Sistema

**Projeto**: TAKTA / FOP — Residência em IA (Instituto Eldorado + UnB/FGA) · Grupo 5
**Versão**: 1.0 · **Data**: 2026-09-29

**Fontes de verdade**: `specs/001-fop-verificacao-alegacoes/contracts/http-api.md` (contrato HTTP), `contracts/verification-result.schema.json` (contrato de dados), `plan.md` (estrutura) e `fluxo-ia-fop-v3.md` (desenho).

**Escopo**: interfaces **externas** do sistema (API HTTP) e **contrato de dados** do resultado. **Sem UI** — telas e fluxos visuais serão definidos pelo fluxo de codificação.

---

## 1. Visão geral

- **Base**: `/api/v1` · **Formato**: `application/json; charset=utf-8`
- **Papéis (MVP)**: `publisher` verificado (`X-Publisher-Id`, conjunto pré-aprovado), `reader` (anônimo), `reviewer` (`X-Reviewer-Id`). `admin` fora do MVP.
- **Modelo assíncrono**: a submissão responde `202` e o cliente consulta o resultado (polling recomendado a cada 5 s; p95 ≤120 s, provisório).
- **Erros**: envelope único `{ "error": { "code": string, "message": string } }`.
- **Confidencialidade (FR-024/FR-025)**: o texto não publicado não sai para serviços externos e é descartado após publicação/conclusão; nenhuma resposta expõe o texto integral após a conclusão.

## 2. Endpoints

| Método | Caminho | Ator | Requisitos |
|--------|---------|------|------------|
| POST | `/api/v1/verifications` | publicador verificado | FR-001, FR-014, FR-016 |
| POST | `/api/v1/verifications/{noticia_id}/versions` | publicador autor | FR-022, FR-023 |
| GET | `/api/v1/verifications/{noticia_id}` | qualquer usuário | FR-017, FR-021 |
| GET | `/api/v1/verifications/{noticia_id}/versions` | publicador/revisor | FR-015, FR-022 |
| GET | `/api/v1/review-queue?status=pendente` | revisor | FR-020 |
| POST | `/api/v1/review-queue/{item_id}/decision` | revisor | FR-018, FR-020, FR-021 |
| GET | `/api/v1/health` | operação | NFR-004, FR-015 |

### 2.1 `POST /api/v1/verifications`

- **Headers**: `X-Publisher-Id: <id>`
- **Body**: `{ "text": "≤1.000 palavras", "title": "...", "language": "pt-BR" }`
- **Respostas**:

| Status | Quando | Corpo |
|--------|--------|-------|
| `202 Accepted` | aceita para processamento | `{ "noticia_id": "...", "versao_texto": 1, "status": "processing" }` |
| `400 Bad Request` | JSON inválido | envelope |
| `403 Forbidden` | publisher desconhecido/revogado | `publisher_not_verified` |
| `422 Unprocessable Entity` | texto vazio, acima de 1.000 palavras ou idioma não suportado | `invalid_text` / `unsupported_language` |

### 2.2 `POST /api/v1/verifications/{noticia_id}/versions`

- **Ator**: publicador autor da notícia. **Limite**: 3 versões (FR-022).
- **Body**: igual ao da submissão.
- **Respostas**: `202` (nova `versao_texto` em `processing`); `409` (`max_versions_reached`); `403`/`404`.
- **Regra**: mudança de veredito de `refuted` para `supported`/`not_enough_evidence` entre versões entra na fila de revisão (FR-023).

### 2.3 `GET /api/v1/verifications/{noticia_id}`

- **Ator**: qualquer usuário (FR-017).
- **Respostas**: `200` processando (`{ "status": "processing" }`); `200` pendente (`{ "status": "pendente_revisao", "review": { "pending_items": N } }`); `200` concluído (`VerificationResult`, inclui `review.publication_allowed`); `404` id desconhecido.

### 2.4 `GET /api/v1/verifications/{noticia_id}/versions`

- Histórico: `{ "versions": [{ "versao_texto", "submitted_at", "status", "changed_verdicts": bool }] }`.

### 2.5 `GET /api/v1/review-queue?status=pendente`

- **Ator**: revisor. Item: `{ item_id, noticia_id, versao_texto, motivo_entrada, evidencias[], diff_versoes, status }`.

### 2.6 `POST /api/v1/review-queue/{item_id}/decision`

- **Ator**: revisor. **Body**: `{ "decisao": "aceitar" | "rejeitar" | "ajustar", "justificativa": "..." }` → `200 { "status": "decidido" }`.

### 2.7 `GET /api/v1/health`

- `{ "status": "ok", "pipeline_version": "...", "snapshots": { "p1": "...", "p2": "...", "config": "..." } }`.

## 3. Contrato de dados — `VerificationResult`

Fonte: `contracts/verification-result.schema.json` (JSON Schema draft-07).

| Campo | Tipo / enum | Obrigatório | Observação |
|-------|-------------|-------------|------------|
| `verification_id` | string | sim | identificador da verificação |
| `noticia_id` | string | sim | identificador da notícia |
| `versao_texto` | inteiro 1–3 | sim | limite de versões (FR-022) |
| `status` | `processing` \| `pendente_revisao` \| `verificada` \| `publicada` \| `error` | sim | mapeamento do status interno (data-model) |
| `submitted_at` / `completed_at` | date-time | sim / não | ISO 8601 |
| `publisher` | `{ id, name }` | sim | publicador responsável |
| `results[]` | array de alegações | sim | uma entrada por alegação principal |
| `results[].claim` | string | sim | alegação descontextualizada |
| `results[].classification` | `supported` \| `refuted` \| `not_enough_evidence` \| `conflicting_evidence` | sim | exatamente uma das 4 classes |
| `results[].origem` | `agencia` \| `fop` | sim | veredito herdado ou decidido pelo fluxo |
| `results[].tipo_atribuicao` | `fonte_citada` \| `declaracao` \| `nenhuma` | sim | a quem o veredito se refere |
| `results[].atribuida_a` / `fonte_citada` | string | não | preenchidos conforme o tipo |
| `results[].sensivel_ao_tempo` / `ambigua` | boolean | não | flags do passo 2 |
| `results[].confidence` | número 0–1 | não | só para origem `fop` e classes decisivas; ausente na inconclusiva |
| `results[].status` | `final` \| `pendente_revisao` | sim | por alegação |
| `results[].motivo` | `agencias_divergem` \| `evidencias_divergem` \| `baixa_confianca` \| `ambigua` \| `numerico_divergente` \| `mudanca_veredito` | não | motivo registrado |
| `results[].evidence[]` | array (≥1) | sim | `excerpt`, `source{name,url}`, `published_at`, `stance` (`favorable`/`contrary`/`reporting`), `decisive` |
| `results[].explanation` | `{ text, generated_by, citations[] }` | sim | `generated_by`: `template` \| `llm`; explicação neutra com citações |
| `results[].uncertainty_note` | string | não | nota de incerteza/conflito |
| `review` | `{ pending_items, publication_allowed }` | sim | portão de publicação (FR-021) |
| `policy` | `{ blocked: false, full_text_retention: "transient" }` | sim | não bloqueio e retenção transitória |

## 4. Códigos de status e erros

| HTTP | `error.code` | Situação |
|------|--------------|----------|
| 400 | — | JSON inválido |
| 403 | `publisher_not_verified` | publicador desconhecido/revogado (FR-016, SC-007) |
| 404 | — | recurso inexistente |
| 409 | `max_versions_reached` | limite de 3 versões (FR-022) |
| 422 | `invalid_text` / `unsupported_language` | texto inválido ou idioma não suportado (FR-014) |

## 5. Regras transversais

- **Papéis**: submissão/ressubmissão só por publicador verificado; leitura por qualquer usuário; fila/decisão só por revisor (FR-018).
- **Revisão**: item pendente ⇒ `review.publication_allowed = false` (FR-021); casos sensíveis listados no FR-020.
- **Temporal**: apenas evidências/checagens até a submissão (FR-012); a idade máxima de checagem é parâmetro (`idade_max_checagem_meses`).
- **Retenção**: `policy.full_text_retention = "transient"`; texto descartado após publicação/conclusão (FR-025).
- **Não bloqueio**: `policy.blocked = false`; notícias refutadas permanecem visíveis após a publicação (FR-010).
- **Auditoria**: `/health` expõe versões de snapshots; cada veredito é rastreável (FR-015).
- **UI fora de escopo**: este documento não define telas; a interface visual será especificada pelo fluxo de codificação.

## 6. Interfaces internas (referência, não vinculante)

Fronteiras sugeridas pelo `plan.md` para o fluxo de codificação (podem ser refinadas):

- `pipeline` (passos 1–7) expõe uma entrada única de verificação e devolve `VerificationResult`;
- `prep/p1_checks`, `prep/p2_evidence` e `prep/p3_calibration` produzem bases/config com `snapshot`;
- `storage` expõe leitura/escrita de vereditos, fila de revisão, auditoria e retenção;
- `retrieval` expõe o serviço de embeddings compartilhado;
- `api` orquestra as chamadas HTTP e a autorização por papel.
