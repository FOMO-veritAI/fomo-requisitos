# Idea Research: TAKTA / FOP — verificação de alegações por IA

- **Slug**: takta
- **Created**: 2026-09-22
- **Evidence confidence (overall)**: medium

> Fonte primária desta pesquisa: o relatório local do próprio grupo, `Relatório Datasets Grupo 5 v2.pdf` (fase *Investigate* da Residência em IA e Análise de Dados — Instituto Eldorado + UnB/FGA). Nenhuma URL externa foi buscada; os dados abaixo derivam do relatório e são citados como tal.

## Users & Demand

- Público-alvo definido no intake: **leitores de notícias em geral** (não restrito a estudantes). — [source: intake.md] (confidence: medium)
- O relatório motiva a demanda por um problema real e atual: desinformação amplificada em redes sociais, com o Brasil como ecossistema relevante (WhatsApp central; eleições 2026; deepfakes/mídia sintética). — [source: Relatório Grupo 5, seções 5.5, 6, 7] (confidence: medium)
- Não há, no material disponível, pesquisa primária com usuários (entrevistas, surveys, dados de uso). A demanda é inferida do contexto, não observada. — [ASSUMPTION] (confidence: low)

## Prior Art

- **AVeriTeC / AVERITEC-BR** — esquema de 4 vereditos (Supported / Refuted / Not Enough Evidence / Conflicting Evidence) idêntico às 4 classes da FOP; versão PT-BR de 2025 elimina tradução própria; kappa 0,619. Reuso direto de protocolo, métrica e baselines. — [source: Relatório Grupo 5, §2.2, §3.1] (confidence: high)
- **CLEF CheckThat! Lab (Tarefa 2, 2025)** — normalização/extração de alegações; 2.183 instâncias em PT. Corresponde à 1ª etapa funcional da FOP. — [source: Relatório Grupo 5, §3.1.3] (confidence: high)
- **Fátima (Aos Fatos)** — checadora conversacional automatizada; concorrente/referência direta do componente; recomenda-se teste comparativo. — [source: Relatório Grupo 5, §7.5.1] (confidence: high)
- **FakeCheck / NILC (Fake.Br)** — detector demonstrativo com ~89% de acurácia em teste controlado, mas saturado por atalhos (comprimento, erros ortográficos); serve de alerta metodológico. — [source: Relatório Grupo 5, §3.2.3] (confidence: high)
- **Projeto Comprova, Coalizão/Fato ou Boato, Agência Lupa, Recod.ai/Horus** — iniciativas operantes de verificação e pesquisa; estrutura editorial do Comprova é esquema de anotação já operante (táticas de persuasão). — [source: Relatório Grupo 5, §7] (confidence: high)

## Market & Context

- Alternativas que os usuários usam hoje: checagem manual em agências (Lupa, Aos Fatos, Comprova), ferramentas conversacionais (Fátima), e desconfiança difusa sem instrumento. — [source: Relatório Grupo 5, §7] (confidence: medium)
- O relatório identifica **lacuna de mercado/pesquisa**: inexistência de corpus brasileiro que associe alegação + evidência recuperada + veredito em 4 classes a partir de conteúdo originalmente brasileiro. — [source: Relatório Grupo 5, §6] (confidence: high)
- Tendência relevante 2026: conteúdo sintético/deepfake em debate público (Projeto VigIA, Horus), ampliando a pertinência do problema. — [source: Relatório Grupo 5, §7.4, §7.5.2] (confidence: medium)

## Data & Constraints

- **Insumo de dados já mapeado e hierarquizado** (18 conjuntos). Topo para a FOP: (1) AVERITEC-BR, (2) AVeriTeC, (3) CheckThat! Tarefa 2, (4) LIAR-BR, (5) FakeRecogna 2/1, (6) FineFake, (7) FactCenter. — [source: Relatório Grupo 5, §4 Quadro 3] (confidence: high)
- Evidência empírica orienta a arquitetura: **o ganho vem da recuperação de evidência, não do tamanho do modelo**; encoders PT-BR ajustados com evidência-ouro superam generativos zero-shot. — [source: Relatório Grupo 5, §3.1.1, §8] (confidence: high)
- **Licenciamento**: AVeriTeC é CC BY-NC 4.0 (impede uso comercial); vários corpora brasileiros não declaram licença formal (risco jurídico se publicados). — [source: Relatório Grupo 5, §3.1.2, §6(item d)] (confidence: high)
- **Riscos metodológicos**: contaminação (LIAR/FEVER em pré-treinamento de LLMs), aprendizado de atalho (Fake.Br), deriva conceitual (detectores estáticos degradam; recomenda-se partição temporal de teste). — [source: Relatório Grupo 5, §5.3] (confidence: high)
- **LGPD/privacidade e moderação**: não tratados no relatório nem definidos no intake. — [NEEDS CLARIFICATION] (confidence: low)

## Evidence Against the Idea

- **Lacuna central de dados**: não existe corpus PT-BR nativo que junte alegação + evidência recuperada + veredito de 4 classes; os multiclasse existentes são traduções automáticas (Gemma 3) cuja distribuição temática difere da brasileira — risco de validade. — [source: Relatório Grupo 5, §6(item a), §3.1.1] (confidence: high)
- **Risco ético/estrutural no coração da proposta**: o MVP decide **bloquear notícias sem revisão humana nem recurso**, o que pode contradizer o próprio princípio de "não substituir o pensamento crítico" e gerar censura indevida. — [source: intake.md (decisão oral do grupo)] (confidence: high)
- **Saturação/atalhos**: conjuntos binários (Fake.Br) inflam acurácia por marcadores superficiais; avaliar neles pode levar a números enganosos. — [source: Relatório Grupo 5, §3.2.3, §5.3] (confidence: high)
- **Contaminação e deriva temporal**: resultados sobre LIAR/FEVER podem refletir memorização; detectores estáticos degradam em notícias correntes (exatamente o caso da FOP pré-publicação). — [source: Relatório Grupo 5, §5.3] (confidence: high)
- **Cobertura e escopo**: só 2 conjuntos cobrem WhatsApp; foco eleitoral de algumas fontes; ausência de corpus rotulado por táticas de persuasão em PT. — [source: Relatório Grupo 5, §6(itens b,c,e)] (confidence: medium)
- **Prazo/recursos da residência**: construir recuperação+verificação com evidência e avaliação rigorosa pode exceder o horizonte de um protótipo acadêmico. — [ASSUMPTION] (confidence: medium)

## Gaps & Open Questions

- [NEEDS CLARIFICATION: composição exata do híbrido generativo + não generativo.]
- [NEEDS CLARIFICATION: mitigação do risco de bloquear sem revisão/recurso, frente ao princípio pedagógico.]
- [NEEDS CLARIFICATION: dataset prioritário para ajuste vs. avaliação no prazo da residência.]
- [NEEDS CLARIFICATION: granularidade do bloqueio — por notícia inteira ou por claim; qual classe bloqueia.]
- [NEEDS CLARIFICATION: LGPD/moderação/licenciamento não comercial.]
- [NEEDS CLARIFICATION: métrica de "estímulo ao pensamento crítico".]

## Sources

- `esboco.txt` (local, no projeto) — esboço da TAKTA.
- `Relatório Datasets Grupo 5 v2.pdf` (local, no projeto) — levantamento de datasets da fase *Investigate*; policy: not-fetched (fonte local, não URL).
- URLs citadas pelo relatório (ex.: github.com, gitlab.com, huggingface.co, thetrustproject.org) **não foram buscadas**; permanecem como referência bibliográfica do próprio relatório.