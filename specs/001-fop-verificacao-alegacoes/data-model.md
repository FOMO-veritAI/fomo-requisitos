# Data Model: FOP — Verificação de alegações por IA

**Feature**: `001-fop-verificacao-alegacoes` | **Date**: 2026-09-27 | **Plan**: [plan.md](./plan.md) | **Spec**: [spec.md](./spec.md) | **Fonte do desenho**: `fluxo-ia-fop-v3.md`

Entidades derivadas da spec (FR-001–FR-025) e do fluxo v3 (P1–P3 e passos 1–7). Texto da notícia tem **retenção transitória** (FR-025).

## Visão geral

```text
P1 Checagens ──┐
P2 Trechos ────┼──> Verificação (notícia/versão) ──> Alegações ──> Casamento ──┬─ agência ─┐
P3 Config ─────┘                                                              └─ fluxo ──┤
                                                                                         ▼
                                              Recuperação → Veredito (NLI+regras) → Calibração/Abstenção
                                                                                         │
                                          ┌──────────────────────────────────────────────┤
                                          ▼                                              ▼
                                   Veredito final                                  Fila de revisão
                                          │                                              │
                                          ▼                                              ▼
                                   Explicação (template)                          Decisão do revisor
                                          │
                                          ▼
                        Resultado ao publicador (pré-publicação) → leitor (pós-publicação)
```

## Entidades

### Publicador verificado
- `id`, `nome`, `tipo` (`veiculo` | `organizacao_editorial`), `status` (`verificado` | `revogado`), `verified_by`, `verified_at`.
- **Regras**: só `verificado` submete (FR-001); revogado → submissão negada.

### Usuário comum
- `id`, `display_name`. Visualiza publicações (FR-017); não submete.

### Revisor
- `id`, `nome`, `papel` (`moderacao` | `veiculo` | `ambos` — decisão da administração).
- **Regras**: decide casos da fila (FR-020); decisão registrada em auditoria.

### Administrador
- `id`, `nome`. Aprova/revoga publicadores (FR-018) — fora do MVP.

### Notícia (retenção transitória)
- `noticia_id`, `versao_texto` (1..3 — FR-022), `veiculo_id`, `titulo`, `texto` (**transitório** — FR-025), `data`, `status`.
- `status` (enum): `em_verificacao` | `pendente_revisao` | `verificada` | `publicada` | `descartada` (mapeamento na API: `em_verificacao` → `processing`; `descartada` → `error`).
- **Regras**: `pendente_revisao` impede publicação (FR-021); texto descartado após publicação/conclusão (FR-025); versões ligadas por `noticia_id`.

### Alegação
- `alegacao_id`, `noticia_id`, `versao_texto`, `frase_original`, `posicao`, `sinais` (número/data/NER/verbo de declaração).
- `alegacao` (descontextualizada), `contexto` (título + frase anterior).
- `entidades[]` (`texto`, `tipo`), `substituicoes[]`, `ambigua` (bool).
- `tipo_atribuicao` (`fonte_citada` | `declaracao` | `nenhuma`), `atribuida_a`, `fonte_citada`.
- `data_referencia`, `sensivel_ao_tempo` (bool).
- `principal` (bool; até 10 por notícia — FR-002).
- **Regras**: `ambigua = true` ⇒ resultado limitado a inconclusiva até revisão (FR-020).

### Checagem de agência (P1)
- `checagem_id`, `alegacao`, `veredito_original`, `classe_fop` (`suportada`|`refutada`|`inconclusiva`|`conflitante`), `motivo` (`agencias_divergem` | nulo), `agencia`, `data`, `url`, `versao_mapeamento`.
- **Regras**: mesma alegação checada por várias agências → grupo; vereditos divergentes → `conflitante`/`agencias_divergem`; guarda-se só alegação curta + metadados + link.

### Casamento
- `alegacao_id`, `checagens[]` (`checagem_id`, `prob_equivalencia`), `origem` (`agencia` | `fop`), `veredito_provisorio`, `motivo`, `filtro_temporal_aplicado` (bool), `checagem_antiga_referencia`.
- **Regras**: só casa com NLI bidirecional acima de `limiar_equivalencia`; alegação sensível ao tempo não herda checagem mais antiga que `idade_max_checagem_meses` (FR-005/FR-012); conflito entre agências → `conflitante`.

### Trecho de evidência (P2)
- `trecho_id`, `doc_id`, `texto` (~150 palavras), `fonte`, `dominio`, `url`, `data_publicacao`, `posicao`.
- **Regras**: nunca vem da própria notícia (FR-006); filtros temporais por `data_publicacao` (FR-012); na avaliação, excluir domínios de agências e documentos posteriores à alegação.

### Veredito
- `alegacao_id`, `veredito_provisorio`, `veredito_final` (4 classes), `origem` (`agencia` | `fop`), `motivo` (`agencias_divergem` | `evidencias_divergem` | `baixa_confianca` | nulo).
- `pontuacao`, `janela_decisiva`, `evidencias[]` (`trecho_id`, `max_p_confirma`, `max_p_contradiz`, `papel`).
- `numerico` (`aplicado`, `resultado`) — incremento.
- `confianca` (só origem `fop` e classes decisivas; inconclusiva **não** recebe — evita circularidade).
- `status` (`final` | `pendente_revisao`), `limiares_aplicados` (`limiar_suporta`, `limiar_refuta`, `limiar_abstencao`).
- **Regras**: ausência de trecho forte ⇒ inconclusiva (FR-008); fortes nos dois sentidos ⇒ conflitante (FR-013); refutada sempre vai para revisão (FR-020).

### Explicação
- `alegacao_id`, `texto` (template no MVP), `fontes[]` (`url`, `fonte`, `data`), `gerado_por` (`template` | `llm`), `status`.
- **Regras**: neutra e com citações (FR-011); por tipo de atribuição ("Declaração de X:", "A notícia atribui este dado a [fonte]:"); LLM é incremento (frases filtradas por NLI; conclusão sempre do template).

### Fila de revisão
- `item_id`, `alegacao_id`, `noticia_id`, `versao_texto`, `motivo_entrada` (`refutada` | `conflitante_pessoa` | `ambigua` | `numerico_divergente` | `mudanca_veredito`), `evidencias[]`, `diff_versoes`, `status` (`pendente` | `decidido`), `revisor_id`, `decisao`, `decidido_em`.
- **Regras**: entrada conforme FR-020/FR-023; publicação suspensa enquanto houver `pendente` (FR-021); decisão registrada em auditoria.

### Registro de auditoria
- `noticia_id`, `versao_texto`, `snapshot_p1`, `snapshot_p2`, `versao_config`, `modelos`, `substituicoes[]`, `tipo_atribuicao`, `resultado`, `timestamps` (submissão/conclusão), histórico de versões ligado.
- **Regras**: imutável; permite reconstruir o veredito (FR-015); sem texto integral após o descarte (FR-025).

### Config / calibradores (P3)
- `versao_config`; parâmetros do `config.json` (ver v3, seção 4) e calibradores salvos em arquivo.
- **Regras**: recalibrar apenas quando um modelo for trocado (ou erro de calibração acima do limite, no piloto); atualização diária das bases **não** recalibra.

## Transições de estado

```text
Notícia: submetida → em_verificacao
   ├─ caso sensível → pendente_revisao → (decisão) → verificada → publicada
   ├─ sem pendências → verificada → publicada
   ├─ publicador corrige → nova versao_texto (≤3) → em_verificacao
   └─ conclusão sem publicação → descartada (texto descartado)

Alegação: selecionada → descontextualizada → casamento
   ├─ casou (agência) ──────────────────────────────┐
   └─ não casou → recuperação → veredito(NLI+regras) ┤
                                                     ▼
                              calibração/abstenção → final | pendente_revisao
                                                     ▼
                              explicação (template) → resultado (publicador/leitor)

Falhas: sem alegação → informa; sem evidência/NLI indisponível → inconclusiva
Usuário não verificado / revogado → negado (sem registro)
```

## Regras transversais (validação)

- **Papéis** (FR-018): submissão só por publisher verificado; leitura por qualquer usuário; revisão por revisor; admin fora do MVP.
- **Temporal** (FR-012): nenhuma evidência/checagem com data posterior à submissão; checagem antiga não é herdada para alegação sensível ao tempo.
- **Retenção** (FR-025): `texto` transitório; somente durante verificação/revisão pendente.
- **Incerteza** (FR-008/FR-013): ausência/falha ⇒ inconclusiva; conflito ⇒ conflitante com motivo; **refutada nunca por falta de evidência**.
- **Revisão** (FR-020/FR-021/FR-023): casos sensíveis bloqueiam publicação até decisão.
- **Exibição** (FR-010/FR-019): simetria favorável × contrária; refutadas visíveis após publicação; nada bloqueado.
- **Confidencialidade** (FR-024): texto não publicado não sai para serviços externos.
