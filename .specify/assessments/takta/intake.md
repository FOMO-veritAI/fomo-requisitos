# Idea Intake: TAKTA — rede social de notícias com verificação por IA

- **Slug**: takta
- **Created**: 2026-09-22
- **Source**: esboco.txt (texto colado) + Relatório Datasets Grupo 5 v2.pdf (neste projeto)
- **Type**: new-capability

## Idea (as captured)

> A TAKTA é uma plataforma social criada para aproximar as pessoas das notícias por meio de interação, crítica, opinião e debate. A proposta é oferecer um ambiente em que os usuários não sejam apenas consumidores passivos de informação, mas participantes ativos das discussões geradas pelas notícias.
>
> Na plataforma, veículos de comunicação e organizações editoriais verificadas poderão publicar conteúdos jornalísticos, enquanto os usuários comuns poderão comentar, compartilhar, reagir e produzir críticas, inclusive em formato de vídeo, sempre relacionadas às notícias já existentes. Dessa forma, cada notícia pode se transformar em um espaço de discussão pública, permitindo que diferentes opiniões, interpretações e perspectivas sejam apresentadas pela comunidade.
>
> Essa dinâmica de interação também é essencial para preservar e estimular o senso crítico dos usuários. A TAKTA não pretende entregar uma interpretação pronta sobre cada notícia, nem substituir a análise humana por uma resposta automática da inteligência artificial. Ao entrar em contato com diferentes críticas, argumentos, reações e pontos de vista, o usuário continua exercitando sua própria capacidade de interpretar, questionar, comparar opiniões e formar conclusões.
>
> A experiência da TAKTA será organizada em três áreas principais. A página de Notícias reunirá as publicações dos veículos acompanhados pelo usuário. A página de Críticas concentrará reações, análises, comentários e vídeos produzidos pela comunidade a partir dessas notícias. Já a página de Populares apresentará os conteúdos com maior repercussão dentro da plataforma, podendo incluir tanto notícias quanto críticas e discussões dos usuários.
>
> Para garantir que esse ambiente de interação seja construído sobre informações mais confiáveis, a TAKTA contará com a FOP — Fact On Publication — uma inteligência artificial responsável por analisar as notícias antes que elas sejam publicadas na plataforma.
>
> A FOP verificará as principais afirmações presentes em uma notícia, buscará evidências externas e comparará essas evidências com o conteúdo submetido. A partir dessa análise, a notícia poderá ser classificada como suportada por evidências, refutada, inconclusiva ou baseada em evidências conflitantes.
>
> Notícias identificadas como falsas ou fortemente contraditas pelas evidências não serão distribuídas normalmente pela plataforma, enquanto conteúdos confiáveis poderão ser publicados e utilizados como base para as interações da comunidade. Em situações em que não houver evidências suficientes, a FOP deverá indicar a incerteza em vez de assumir automaticamente que a informação é falsa.
>
> Assim, o combate à fake news acontece como consequência da própria estrutura da TAKTA: antes que uma notícia se torne objeto de comentários, críticas, compartilhamentos e debates, ela passa por uma etapa de verificação. Isso reduz a possibilidade de que informações falsas sejam amplificadas pela comunidade e permite que as discussões ocorram a partir de uma base informacional mais confiável.
>
> Ao mesmo tempo, a existência dessa verificação prévia não elimina o papel das pessoas na interpretação da informação. A FOP atua sobre a confiabilidade factual da notícia, enquanto a comunidade continua responsável por discutir seus significados, impactos, consequências e diferentes perspectivas. Dessa forma, a inteligência artificial oferece suporte ao combate à desinformação sem retirar do usuário sua autonomia de pensamento.
>
> O objetivo da TAKTA, portanto, é criar uma rede social de notícias em que a tecnologia contribua para a confiabilidade da informação e a comunidade contribua para o desenvolvimento do pensamento crítico. A FOP ajuda a proteger o ambiente contra fake news, enquanto as interações entre os usuários mantêm viva a capacidade de questionar, argumentar, interpretar e construir opiniões próprias.

## Restated

Plataforma social de notícias onde veículos verificados publicam conteúdo e a comunidade comenta, critica e debate (texto e vídeo); uma IA, a FOP — Fact On Publication — analisa as principais afirmações de cada notícia contra evidências externas **antes da publicação**, classificando-a como suportada, refutada, inconclusiva ou com evidências conflitantes, sem substituir a interpretação humana. **Decisão de recorte atual (2026-09-22): o esforço da residência concentra-se no componente FOP**, como protótipo acadêmico/MVP demonstrável, para leitores de notícias em geral; o restante da plataforma TAKTA permanece como contexto e trabalho futuro.

## Origin & Context

- **Raised by**: Grupo 5 da Residência em Inteligência Artificial e Análise de Dados (Instituto Eldorado + UnB/FGA — Campus Gama) — Cauan Soares, Isabel Azar, Maria Gladys, Mickeias Charles. **Challenge 1**: como sistemas de IA podem auxiliar pessoas a avaliar a confiabilidade da informação sem substituir seu pensamento crítico. A resposta do grupo é a TAKTA / FOP.
- **Trigger**: projeto de residência voltado a alunos de graduação da área de TI; tema central é combater a desinformação sem substituir o pensamento crítico.
- **Contexto adicional**: já existe um relatório de levantamento de datasets ("Relatório Datasets Grupo 5 v2"), produzido na fase *Investigate*, que hierarquiza 18 conjuntos de dados públicos por utilidade à FOP, aponta lacunas e recomenda uma arquitetura de dados em três camadas (extração de alegações → recuperação e verificação → avaliação de fonte).

## Clarifications Resolved (2026-09-22)

- **Usuários-alvo**: público geral / leitores de notícias (não restrito a estudantes).
- **Escopo**: protótipo acadêmico / MVP demonstrável, avaliado ao final da residência.
- **Recorte**: apenas a **FOP** (verificação por IA), não a plataforma completa.
- **Métricas de sucesso**: qualidade/acurácia da FOP, demonstração funcional do MVP e estímulo ao pensamento crítico.
- **Insumo de dados**: usar os datasets levantados no relatório do Grupo 5; direção preliminar é um modelo **não generativo** (ou híbrido: não generativo + generativo), explorando recuperação de evidência, não apenas prompt engineering.
- **Superfície do MVP**: interface web mínima (publicador submete notícia → ver classificação + evidências; usuário comum visualiza).
- **Publicação e papéis (revisão de 2026-09-27)**: apenas **publicadores verificados** (veículos/organizações aprovados pela administração da plataforma) podem submeter/publicar notícias; **usuários comuns apenas visualizam e interagem**. Coerente com o esboço original (`esboco.txt`), que previa veículos verificados publicando e a comunidade comentando/criticando.
- **Política de conteúdo (revisada em 2026-09-22, após discussão do time)**: a plataforma **não bloqueia nem oculta** notícias classificadas como desinformação. Notícias falsas/refutadas permanecem visíveis **acompanhadas da classificação e dos argumentos/evidências que a sustentam**, justamente como estímulo ao pensamento crítico. Esta decisão substitui o entendimento anterior de bloqueio sem revisão.
- **Princípio de design para pensamento crítico**: a IA não entrega veredito seco; apresenta classificação + evidências para o usuário formar sua própria conclusão.
- **Arquitetura**: direção desejada é **aliar IA generativa e não generativa** (ex.: recuperação não generativa com apoio generativo); composição exata ainda em aberto.
- **Arquitetura definitiva (2026-09-27)**: `fluxo-ia-fop-v3.md` define o plano do grupo — seleção de frases por regras+spaCy; descontextualização/atribuição; casamento com checagens de agências (embedding + NLI); recuperação BM25+densa+RRF; **veredito por NLI em janelas + regras (sem LLM)**; calibração/abstenção; **fila de revisão humana**; explicação por templates. O anexo técnico anterior fica superado.

## First-Glance Unknowns

- [NEEDS CLARIFICATION: composição exata do híbrido generativo + não generativo — o que cada parte faz? Requer discussão (candidato ao stage `shape`).]
- [NEEDS CLARIFICATION: como apresentar a classificação e as evidências sem enviesar a leitura (ordem, destaque, linguagem) e sem amplificar a desinformação.]
- [NEEDS CLARIFICATION: qual(is) dataset(s) prioritário(s) para ajuste vs. avaliação, dado o orçamento de tempo da residência?]
- [NEEDS CLARIFICATION: dados/privacidade (LGPD), licenciamento não comercial do AVeriTeC e moderação de conteúdo.]
- [NEEDS CLARIFICATION: como o MVP demonstra "estímulo ao pensamento crítico" de forma mensurável, além do princípio de design?]