# Concept: FOP — verificação de alegações por IA em PT-BR

- **Slug**: takta
- **Created**: 2026-09-22
- **Recommended option**: Opção B — Híbrido em camadas (recuperação não generativa + veredito por encoder PT-BR + generativo para extração/explicação)
- **Nota (2026-09-27)**: arquitetura superada pelo `fluxo-ia-fop-v3.md` (veredito por NLI+regras, sem LLM; casamento com checagens; revisão humana; explicação por templates). Mantido como registro histórico do estágio `shape`.

## Options

### Opção A — Menor coisa que funciona: recuperação simples + classificador supervisionado
- **Sketch**: A notícia é quebrada em alegações por regras/sentenças; cada alegação é buscada (BM25) contra uma base de checagens brasileiras já verificadas (FactCenter) e contra o repositório de evidência do AVeriTeC; um encoder PT-BR ajustado no AVERITEC-BR decide uma das 4 classes; a interface mostra classe + trechos de evidência recuperados, sem texto gerado. O usuário vê o contraste, não uma explicação redigida.
- **Appetite**: medium (semanas)
- **Trade-offs**: **Ganha** viabilidade, determinismo e aderência exata à evidência do relatório (o ganho vem da recuperação, não do tamanho do modelo); entrega métricas comparáveis a baselines publicados. **Sacrifica** qualidade da extração de alegações (sem generativo, a segmentação é frágil) e qualidade da explicação ao leitor (trechos crus, menos didáticos).
- **Rabbit holes**: alinhamento de domínio/idioma da base de evidência (AVeriTeC é em inglês); qualidade da segmentação de alegações em notícias longas; licença NC do AVeriTeC; partição temporal de teste.

### Opção B — Híbrido em camadas (recomendada)
- **Sketch**: Três camadas conceituais alinhadas ao relatório. (1) **Extração**: um modelo generativo identifica e normaliza as alegações verificáveis da notícia, com saída estruturada — inspirado na tarefa do CheckThat!. (2) **Recuperação e veredito**: busca não generativa (léxica + densa + reranking) recupera evidência das bases; um encoder PT-BR ajustado com evidência-ouro emite o veredito nas 4 classes. (3) **Explicação**: o generativo, estritamente ancorado na evidência recuperada e com citações, redige a explicação que o leitor vê — a IA **não dá veredito seco** e a interface exibe igualmente refutadas e inconclusivas com seus argumentos, como estímulo ao pensamento crítico.
- **Appetite**: large (meses) — com degradação planejada para a Opção A se o prazo apertar
- **Trade-offs**: **Ganha** o melhor equilíbrio entre acurácia baseada em evidência (veredito ancorado) e legibilidade para o usuário (explicação generativa), atendendo à intenção do grupo de aliar IA generativa e não generativa sem entregar o veredito ao LLM. **Sacrifica** simplicidade e tempo de engenharia; exige orquestrar 3 componentes e cuidar de alucinação na camada de explicação.
- **Rabbit holes**: grounding da explicação (evitar que o generativo contradiga o veredito do classificador); custo/latência de LLM; decidir extração com LLM local vs. API; construir/avaliar a classe de evidências conflitantes (rara); calibração da abstenção.

### Opção C — Pipeline generativo com RAG (o LLM decide)
- **Sketch**: Um único LLM faz extração, consulta a base via RAG e emite o veredito com justificativa. Prompt engineering como núcleo.
- **Appetite**: small (dias a poucas semanas)
- **Trade-offs**: **Ganha** velocidade de prototipagem e explicações naturalmente fluidas. **Sacrifica** a evidência do próprio relatório: LLMs têm dificuldade em verificação zero-shot, o veredito não fica ancorado em recuperação estruturada e há risco de alucinação — potencialmente contradizendo a premissa de confiabilidade da FOP.
- **Rabbit holes**: veredito não reproduzível; dificuldade de medir acurácia contra baselines; contaminação (o LLM pode ter visto os datasets); explicar a classificação sem evidência rastreável.

### Opção D — Não construir / reutilizar Fátima (do nothing / buy)
- **Sketch**: Não desenvolver a FOP; usar a checadora conversacional Fátima (Aos Fatos) ou serviços de fact-checking existentes e concentrar o esforço na avaliação comparativa.
- **Appetite**: small
- **Trade-offs**: **Ganha** custo mínimo e um benchmark real pronto. **Sacrifica** o objetivo da residência (construir o componente), a contribuição original (lacuna brasileira identificada) e o controle sobre a experiência de exibição de classificações/evidências. Faz sentido apenas como **referência comparativa**, não como plano.
- **Rabbit holes**: dependência de terceiros; sem código/controle sobre a classificação; não atende ao Challenge 1 como entrega técnica.

## Recommendation

**Opção B — Híbrido em camadas**, com a Opção A como *fallback* estruturado caso o tempo não permita as três camadas.

Razão, atada aos goals e métricas: o relatório do grupo demonstra que o desempenho vem da **recuperação de evidência, não do tamanho do modelo**, e que encoders PT-BR ajustados com evidência-ouro superam generativos zero-shot. Manter o **veredito** numa camada não generativa ancorada em evidência protege a confiabilidade e a mensurabilidade (macro-F1 por classe, refutada vs. inconclusiva, conflitante, abstenção calibrada). Usar o **generativo apenas onde ele agrega** — extração de alegações e redação da explicação ancorada — atende à intenção do grupo de aliar as duas famílias de IA sem entregar o veredito ao LLM, e viabiliza o princípio "a IA não dá veredito seco". A exibição de notícias refutadas com seus argumentos é diretamente suportada pela camada de explicação.

**Nota sobre detalhamento técnico**: por restrição do estágio `shape` (que não produz arquitetura), o plano técnico sólido com ferramentas, avaliação e cronograma que o grupo pediu está registrado em anexo separado: `concept-annex-tecnico.md`. Ele informa a recomendação, mas não é o artefato de decisão.

## Out of Scope (for the recommended option)

- Plataforma TAKTA completa (Notícias / Críticas / Populares). — herdado de problem.md
- Bloqueio ou ocultação de notícias; veredito binário; substituir jornalistas/agências.
- Verificação multimodal (imagem/vídeo) no MVP.
- Revisão humana/recurso formal e uso comercial (licença NC).
- Verificação/aprovação de publicadores (administração da plataforma) e interações sociais (comentários/críticas). — [revisão 2026-09-27]
- Treinar um LLM do zero ou pesquisa de arquitetura de foundation models.

## Assumptions to Validate

- Acesso e licença do AVERITEC-BR/LIAR-BR junto aos autores (tradução automática; uso de anotação de terceiros) e disponibilidade do FactCenter.
- Recursos de GPU (ou créditos) para ajustar encoders PT-BR; orçamento para LLM via API, ou viabilidade de LLM local.
- Prazo real da residência (ainda não registrado) compatível com o appetite `large`; se não, ativar degradação para a Opção A.
- Disponibilidade de partição temporal de teste (ex.: Fato ou Boato) para medir deriva conceitual.