# Decision: FOP — verificação de alegações por IA em PT-BR

- **Slug**: takta
- **Decided**: 2026-09-27
- **Verdict**: go
- **Artifacts reviewed**: intake.md sim | research.md sim | problem.md sim | concept.md sim

## Scorecard

| Criterion | Rating | Justification |
|-----------|--------|---------------|
| Problem validity | strong | Problema real e documentado: a desinformação circula antes da checagem profissional e o leitor não tem meio rápido, transparente e baseado em evidências para julgar notícias recém-publicadas (`problem.md`; `research.md`). |
| Evidence strength | adequate | Evidência secundária robusta e verificada — relatório do Grupo 5 com 18 conjuntos hierarquizados, baselines publicados e riscos mapeados; confiança geral `medium`; sem pesquisa primária com usuários. |
| Value vs. inaction | adequate | O MVP entrega um instrumento de verificação ancorado em evidências e habilita um corpus PT-BR inédito; sem ele, resta somente o levantamento documental (`problem.md` — Custo de Inação). |
| Feasibility / appetite | adequate | Opção B (híbrida em camadas) com appetite `large` e degradação planejada para a Opção A; apenas um componente exige treino; infraestrutura acessível em GPU gratuita (`concept.md`; anexo técnico). |
| Strategic fit | strong | Alinha-se diretamente ao Challenge 1 da residência — IA que apoia o pensamento crítico sem substituí-lo. Constitution ainda não ratificada, portanto não avaliada. |
| Risk posture | adequate | Riscos principais identificados e mitigados (contaminação, aprendizado de atalho, deriva temporal, alucinação na explicação, licença NC, infraestrutura). Riscos abertos: licença não comercial, LGPD e métrica de pensamento crítico. |

## Verdict & Rationale

**Go.** A validade do problema é `strong` e a força de evidência atinge o mínimo exigido (`adequate`), com conceito recomendado e escopo delimitado. As incertezas remanescentes são de **planejamento e medição**, não de viabilidade — e seguem registradas como perguntas em aberto no handoff, para resolução no planejamento e não como bloqueio. O veredito autoriza o `speckit.specify` para o recorte FOP.

## If go — Handoff to `speckit.specify`

- **Problem**: leitores de notícias não têm um meio rápido, transparente e baseado em evidências para julgar se as afirmações de uma notícia recém-publicada são factualmente sustentáveis.
- **Chosen approach**: Opção B — híbrido em camadas (extração de alegações por IA generativa; recuperação de evidência não generativa; veredito em 4 classes por modelo supervisionado; explicação ancorada + interface web mínima), com fallback para a Opção A (sem camadas generativas).
- **In scope / out of scope**: MVP da FOP em PT-BR (interface web mínima), com **submissão/ressubmissão restrita a publicadores verificados**, usuários comuns como leitores, **fila de revisão humana** para casos sensíveis e resultado por alegação (4 classes) com evidências e explicação neutra; refutadas permanecem visíveis (não bloqueia nem oculta). Fora do escopo: plataforma TAKTA completa (Notícias/Críticas/Populares), aprovação/revogação de publicadores (administração), canal formal de contestação, interações sociais, verificação multimodal, uso comercial. **Arquitetura realinhada ao `fluxo-ia-fop-v3.md` (2026-09-27)**: regras+spaCy, casamento com checagens, recuperação híbrida, veredito por NLI+regras (sem LLM), calibração/abstenção e explicação por templates.
- **Success metrics**: desempenho por classe (separando refutada de inconclusiva), desempenho na classe de evidências conflitantes, taxa de abstenção calibrada, demonstração funcional ponta a ponta e avaliação de estímulo ao pensamento crítico.
- **Carried-forward open questions**: composição exata do híbrido generativo + não generativo; como apresentar classificação e evidências sem enviesar a leitura; dataset prioritário para ajuste vs. avaliação; LGPD e licenciamento não comercial; métrica de "estímulo ao pensamento crítico"; critérios e fluxo de verificação de publicadores pela administração.
