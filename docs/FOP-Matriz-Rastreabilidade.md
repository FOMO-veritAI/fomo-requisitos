# FOP — Matriz de Rastreabilidade

**Projeto**: TAKTA / FOP — Residência em IA (Instituto Eldorado + UnB/FGA) · Grupo 5
**Versão**: 1.0 · **Data**: 2026-09-29

**Fontes**: `spec.md` (US/FR/SC), `plan.md` (NFR), `tasks.md` (T001–T067), `docs/FOP-Interfaces-Sistema.md` (interfaces), `docs/FOP-Requisitos-Detalhados.md` (RF/RNF granulares) e `fluxo-ia-fop-v3.md` (passos).
**Como ler**: cada linha liga um requisito às tarefas que o implementam e ao ponto do fluxo v3 correspondente.

---

## 1. Requisitos de usuário → requisitos de sistema → critérios de sucesso

| US | Prioridade | FRs relacionados | NFRs relacionados | SCs | Passos do v3 |
|----|-----------|------------------|-------------------|-----|--------------|
| US1 — Publicador submete e recebe a verificação | P1 🎯 | FR-001..FR-016, FR-018, FR-020..FR-025 | NFR-001..NFR-009, NFR-012..NFR-014 | SC-001, SC-005, SC-007, SC-008, SC-009, SC-010 | P1–P3; passos 1–7 |
| US2 — Leitor vê publicadas sem bloqueio | P2 | FR-010, FR-017, FR-018, FR-019 | NFR-004, NFR-010, NFR-011, NFR-013 | SC-004, SC-006 | passo 7 (entrega) |
| US3 — Conferir fontes, atribuição e incerteza | P3 | FR-003, FR-009, FR-011, FR-013 | NFR-006, NFR-011 | SC-002 | passos 2 e 7 |
| US4 — Revisor decide casos pendentes | P3 | FR-018, FR-020, FR-021, FR-022, FR-023 | NFR-006, NFR-010 | SC-008, SC-010 | passo 6.3 |

---

## 2. Requisitos funcionais → tarefas → passo do v3

| FR | Resumo | Tarefas | Passo v3 |
|----|--------|---------|----------|
| FR-001 | Submissão só por publicador verificado | T011, T020, T035 | 1 (pré-condição) |
| FR-002 | Seleção de frases verificáveis (≤10) | T022, T025 | 1 |
| FR-003 | Descontextualização e atribuição | T022, T026 | 2 |
| FR-004 | Sensibilidade ao tempo | T026 | 2 (C7) |
| FR-005 | Casamento com checagens existentes | T022, T027 | 3 |
| FR-006 | Recuperação de evidência (lexical + semântica) | T023, T029 | 4 |
| FR-007 | Veredito em 4 classes | T023, T030 | 5 |
| FR-008 | Ausência de evidência ≠ falsidade | T030, T034 | 5 |
| FR-009 | Evidências com fonte, data e trecho decisivo | T047 | 7 |
| FR-010 | Refutadas visíveis (sem bloqueio) | T044, T045 | 7 (entrega) |
| FR-011 | Explicação neutra com atribuição | T033, T048 | 7 |
| FR-012 | Corte temporal (submissão; idade de checagem) | T027, T029, T061 | 3/4/6 |
| FR-013 | Declarar incerteza/conflito com motivo | T031, T049 | 5/6 |
| FR-014 | Operar em português do Brasil | T024 | 1 (detecção) |
| FR-015 | Registro de auditoria (snapshots, versões) | T009, T039 | 6.4 |
| FR-016 | Negar submissão de não verificados | T011, T020 | 1 |
| FR-017 | Usuários comuns visualizam | T042 | entrega |
| FR-018 | Distinção de papéis | T011, T052 | 6 |
| FR-019 | Apresentação simétrica (favorável × contrária) | T045 | 7 |
| FR-020 | Fila de revisão para casos sensíveis | T032, T052 | 6.3 |
| FR-021 | Bloqueio de publicação com pendência | T037, T051 | 6.3 |
| FR-022 | Ressubmissão (até 3 versões) | T020, T036 | 6.3 (C4) |
| FR-023 | Mudança de veredito → revisão | T032, T036 | 6.3 (C4) |
| FR-024 | Texto não publicado não sai para terceiros | T062 | 4.3/7.3 (W12) |
| FR-025 | Retenção transitória do texto | T010 | P1/6.4 |

---

## 3. Requisitos não funcionais → tarefas → verificação

| NFR | Resumo | Tarefas | Verificação |
|-----|--------|---------|-------------|
| NFR-001 | Latência p95 ≤120 s (notícia típica) | T035, T059 | Benchmark E9; SC-005 |
| NFR-002 | Orçamento por etapa | T013, T059 | Logs de tempo; E9 |
| NFR-003 | Concorrência 10 / ~50 dia / pico 10× | T059 | E9 |
| NFR-004 | Disponibilidade ≥95% na avaliação | T066 | Smoke test do deploy |
| NFR-005 | Falha de componente não é fatal | T034 | Testes de integração com falhas |
| NFR-006 | Observabilidade (logs JSON, métricas) | T013, T056, T058 | Métricas p50/p95, abstenção, fila |
| NFR-007 | Auditabilidade/reprodutibilidade | T009, T018, T039 | Snapshots + seeds; registro imutável |
| NFR-008 | Confidencialidade do texto | T005, T062 | Revisão de chamadas externas |
| NFR-009 | Privacidade/LGPD | T010, T063 | Retenção + checklist jurídico |
| NFR-010 | Acessibilidade WCAG 2.1 AA | T057 | Auditoria de acessibilidade |
| NFR-011 | Usabilidade/transparência | T045, T048, T049 | Revisão de UX + SC-006 |
| NFR-012 | Manutenibilidade (cobertura ≥70%) | T003, T067 | `pytest --cov` + gate em CI |
| NFR-013 | Portabilidade (sem GPU; CPU) | T066 | Execução em CPU/contêiner |
| NFR-014 | Custo operacional ~zero | T028, T066 | Cache + demo CPU |

---

## 4. Critérios de sucesso → requisitos → tarefas

| SC | Requisitos ligados | Tarefas |
|----|--------------------|---------|
| SC-001 | FR-007, FR-008 | T030, T034 |
| SC-002 | FR-009 | T047 |
| SC-003 | FR-007 | T018, T060 |
| SC-004 | FR-010 | T044, T045 |
| SC-005 | NFR-001 | T059 |
| SC-006 | NFR-011 | T045, T049 (verificação por enquete pós-uso, fora do build) |
| SC-007 | FR-001, FR-016 | T011, T020 |
| SC-008 | FR-020, FR-021 | T032, T037 |
| SC-009 | FR-024 | T062 |
| SC-010 | FR-022, FR-023 | T036 |

---

## 5. Requisitos → interfaces de sistema

| Requisito | Interface / operação |
|-----------|----------------------|
| FR-001, FR-016 | `POST /api/v1/verifications` + `403 publisher_not_verified` |
| FR-003, FR-011 | `results[].tipo_atribuicao`, `atribuida_a`, `fonte_citada`, `explanation` |
| FR-007, FR-008, FR-013 | `results[].classification`, `motivo`, `uncertainty_note` |
| FR-009 | `results[].evidence[]` (`excerpt`, `source`, `published_at`, `decisive`) |
| FR-010, FR-019 | `policy.blocked = false`; `stance` (`favorable`/`contrary`) |
| FR-012 | filtro temporal interno; parâmetro `idade_max_checagem_meses` (config) |
| FR-014, FR-024, FR-025 | `422 unsupported_language`; `policy.full_text_retention = "transient"`; sem texto nas respostas |
| FR-015 | `GET /api/v1/health` (snapshots) e `GET /verifications/{id}/versions` |
| FR-017 | `GET /api/v1/verifications/{noticia_id}` |
| FR-018 | headers `X-Publisher-Id`/`X-Reviewer-Id`; endpoints de revisão |
| FR-020 | `GET /api/v1/review-queue` |
| FR-021 | `review.publication_allowed`; `POST /review-queue/{item_id}/decision` |
| FR-022, FR-023 | `POST /verifications/{noticia_id}/versions`; `GET /versions` |
| NFR-001 | `POST` responde `202`; polling em `GET` (p95 ≤120 s) |
| NFR-004 | `GET /api/v1/health` (deploy/smoke test) |
| NFR-008, NFR-009 | sem texto em chamadas externas; `policy`; retenção transitória |

## 6. Cobertura e lacunas

- **FRs**: 25/25 com tarefa (100%). **NFRs**: 14/14 com tarefa (100%). **SCs**: 9/10 no build — SC-006 é métrica de resultado pós-uso (enquete), verificada na avaliação com usuários.
- Tarefas de **Setup/Fundação/Polish** que não aparecem nas tabelas são de suporte (ex.: T001–T008, T012, T014–T017, T019, T021, T040–T041, T046, T050–T051, T053–T055, T057–T059, T061–T067).
- **Provisório**: NFR-001/NFR-002 até o benchmark E9 e o reajuste do piloto E11.
- **Decisões abertas do v3** (§8) não bloqueiam a matriz, mas podem gerar novas tarefas: exibição de "inconclusiva", canal de contestação, precisão-alvo de refutada e parecer jurídico.

## 7. Grupos granulares (RF/RNF) → FR/NFR → tarefas

Mapeamento por grupo do `docs/FOP-Requisitos-Detalhados.md` (sem inflar com 132 linhas individuais).

| Grupo granular | RF/RNF | FR/NFR correspondentes | Tarefas |
|----------------|--------|------------------------|---------|
| 1. Papéis e acesso | RF-001..007 | FR-001, FR-016, FR-017, FR-018 | T011, T020, T035, T042, T052 |
| 2. Submissão e versionamento | RF-008..018 | FR-001, FR-014, FR-015, FR-022 | T020, T024, T035, T036 |
| 3. Alegações (seleção, atribuição, tempo) | RF-019..031 | FR-002, FR-003, FR-004, FR-020 | T022, T025, T026 |
| 4. Casamento com checagens | RF-032..038 | FR-005, FR-012, FR-013, NFR-014 | T022, T027, T028 |
| 5. Recuperação de evidência | RF-039..046 | FR-003, FR-006, FR-012 | T023, T029 |
| 6. Veredito | RF-047..058 | FR-007, FR-008, FR-009, FR-013 | T023, T030 |
| 7. Calibração, abstenção e revisão | RF-059..071 | FR-013, FR-015, FR-020, FR-021, FR-023 | T031, T032, T037, T052, T053, T054 |
| 8. Explicação e exibição | RF-072..085 | FR-009, FR-010, FR-011, FR-013, FR-014, FR-019 | T033, T045, T047, T048, T049 |
| 9. Auditoria, retenção e confidencialidade | RF-086..095 | FR-015, FR-024, FR-025, NFR-008, NFR-009 | T009, T010, T039, T062 |
| 10. Preparação das bases (P1–P3) | RF-096..106 | v3 P1–P3 (sem FR direto) | T016, T017, T018 |
| 11. Desempenho e capacidade | RNF-001..005 | NFR-001..NFR-003 | T035, T059 |
| 12. Disponibilidade e confiabilidade | RNF-006..009 | NFR-004, NFR-005 | T034, T066 |
| 13. Observabilidade e auditabilidade | RNF-010..013 | NFR-006, NFR-007 | T013, T056, T058 |
| 14. Segurança, privacidade e conformidade | RNF-014..018 | NFR-008, NFR-009 | T005, T010, T062, T063 |
| 15. Usabilidade, acessibilidade e qualidade | RNF-019..026 | NFR-010..NFR-014 | T003, T028, T057, T066, T067 |
