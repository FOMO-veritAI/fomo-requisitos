# Contrato HTTP — FOP API

**Feature**: `001-fop-verificacao-alegacoes` | **Date**: 2026-09-27 | **Fonte do desenho**: `fluxo-ia-fop-v3.md`

Superfície externa do MVP: submissão/ressubmissão (publicador verificado), leitura (usuário comum) e fila de revisão (revisor). Resultado segue `verification-result.schema.json`.

## Convenções

- Base: `/api/v1`
- Formato: `application/json; charset=utf-8`
- Erros: envelope `{ "error": { "code": string, "message": string } }`
- Papéis (MVP): `publisher` verificado (`X-Publisher-Id`, conjunto pré-aprovado), `reader` anônimo, `reviewer` (`X-Reviewer-Id`). `admin` fora do MVP.
- Processamento assíncrono: submissão responde `202`; o cliente consulta o resultado (p95 ≤120 s, provisório).
- **Confidencialidade**: o texto não é enviado a serviços externos e é descartado após publicação/conclusão (FR-024/FR-025).

## `POST /api/v1/verifications`

Submete uma notícia. **Somente publicador verificado** (FR-001).

**Headers**: `X-Publisher-Id: <id>`

**Body**: `{ "text": "≤1.000 palavras", "title": "...", "language": "pt-BR" }`

| Status | Quando | Corpo |
|--------|--------|-------|
| `202 Accepted` | aceita | `{ "noticia_id": "...", "versao_texto": 1, "status": "processing" }` |
| `400` | JSON inválido | envelope |
| `403` | publisher desconhecido/revogado (FR-016, SC-007) | `publisher_not_verified` |
| `422` | texto vazio/>>1.000 palavras/idioma não suportado | `invalid_text` / `unsupported_language` |

## `POST /api/v1/verifications/{noticia_id}/versions`

Ressubmissão (correção) da mesma notícia. **Somente o publicador autor**. Limite: 3 versões (FR-022).

**Body**: igual ao POST de submissão.

| Status | Quando |
|--------|--------|
| `202 Accepted` | nova `versao_texto` criada (`processing`) |
| `409 Conflict` | limite de 3 versões atingido (`max_versions_reached`) |
| `403` / `404` | não autor / notícia inexistente |

- Mudança de veredito de **refutada** para **suportada/inconclusiva** entre versões entra na fila de revisão (FR-023).

## `GET /api/v1/verifications/{noticia_id}`

Consulta o resultado. **Qualquer usuário** (FR-017).

| Status | Quando | Corpo |
|--------|--------|-------|
| `200` | processando | `{ "status": "processing" }` |
| `200` | pendente de revisão | `{ "status": "pendente_revisao", "review": { "pending_items": N } }` |
| `200` | concluído | `VerificationResult` (ver schema; inclui `publication_allowed`) |
| `404` | id desconhecido | envelope |

**Polling recomendado**: a cada 5 s; após 120 s sem conclusão, estado de espera/erro.

## `GET /api/v1/verifications/{noticia_id}/versions`

Histórico de versões (publisher/reviewer): `{ "versions": [{ "versao_texto", "submitted_at", "status", "changed_verdicts": bool }] }`.

## `GET /api/v1/review-queue?status=pendente`

Lista a fila de revisão. **Somente revisor** (FR-020). Cada item: `{ item_id, noticia_id, versao_texto, motivo_entrada, evidencias[], diff_versoes, status }`.

## `POST /api/v1/review-queue/{item_id}/decision`

Registra a decisão do revisor. **Somente revisor** (FR-018/FR-020).

**Body**: `{ "decisao": "aceitar" | "rejeitar" | "ajustar", "justificativa": "..." }` → `200 { "status": "decidido" }`.

- Enquanto houver item `pendente`, `publication_allowed = false` (FR-021).

## `GET /api/v1/health`

`{ "status": "ok", "pipeline_version": "...", "snapshots": { "p1": "...", "p2": "...", "config": "..." } }`.

## Regras de exibição (FR-010/FR-011/FR-019)

O `VerificationResult` entrega, por alegação:
- exatamente uma `classification` (4 valores) e `origem` (`agencia` | `fop`);
- `tipo_atribuicao` (`fonte_citada` | `declaracao` | `nenhuma`) com `atribuida_a`/`fonte_citada`, para o leitor saber a quem o veredito se refere;
- `evidence[]` com `excerpt`, `source` (nome/url), `published_at` e `stance` (`favorable` | `contrary` | `reporting`) para o layout simétrico;
- `janela_decisiva` (trecho que sustenta o veredito, limitado a ~25 palavras);
- `explanation.text` neutra com citações, sem prescrever conclusão;
- `policy.blocked = false` e `policy.full_text_retention = "transient"`.

Nenhuma resposta expõe o texto integral submetido após a conclusão (FR-025).
