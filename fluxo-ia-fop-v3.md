# Fluxo de IA da FOP — versão 3 (plano definitivo)

*Grupo 5 – Challenge 1 – Residência em IA (Instituto Eldorado / UnB Gama) · 25/09/2026*

**Finalidade:** para cada notícia submetida por um veículo verificado, entregar **antes da publicação na TAKTA** um resultado **por alegação**. Cada resultado diz se a alegação é suportada, refutada, inconclusiva ou conflitante, e vem acompanhado das evidências usadas e de uma explicação conferida. O objetivo é que o veículo possa corrigir a notícia e que o leitor avalie a informação por conta própria, sem que a IA decida por ele. **Uma declaração que o veículo apenas relata é avaliada como declaração de quem a fez; um dado que o veículo atribui a uma fonte continua sendo responsabilidade do veículo.**

**Estado atual:** plano definitivo, a caminho de protótipo. Nenhum componente foi medido ainda. Os números obtidos no protótipo **não estimam o desempenho em produção**; isso só é medido no piloto (seção 6, item 7).

---

## 0. Como ler este documento

O fluxo tem duas fases:

- **Preparação (P1 a P3):** roda antes de o sistema entrar no ar e depois é atualizada periodicamente. P3 roda **depois** de os passos 1 a 6 estarem montados com valores padrão, e alimenta os passos 3, 5 e 6.
- **Execução (1 a 7):** roda a cada notícia submetida.

Cada passo segue a mesma ordem: 1. Identificação e o que faz; 2. O que entra; 3. Como processa; 4. Com que ferramentas; 5. O que sai; 6. Contribuição para a finalidade; 7. Como verificar.

**MVP e incrementos.** Cada passo indica o que entra no **MVP**, a menor versão que roda de ponta a ponta e pode ser avaliada, e o que é **incremento**. Um incremento só entra depois que uma ablação (medir o fluxo com e sem ele) mostrar ganho.

**Códigos.** W# remete às fraquezas do primeiro parecer de crítica; N# e C# aos problemas e correções do parecer sobre a versão 2; E# aos experimentos da seção 7. As licenças citadas devem ser conferidas antes de qualquer uso comercial.

### 0.1 O que mudou na versão 2

| Crítica | Decisão | Onde |
|---|---|---|
| W1 Vazamento na avaliação | Protocolo anti-vazamento: excluir domínios de agências e documentos posteriores à alegação; deduplicar o teste contra P1; avaliar o passo 3 à parte. Filtro temporal depende de o knowledge store trazer data por documento (a confirmar na EDA) | Seção 6, P2, P3, passo 4 |
| W2 Direito autoral | P2 no MVP só com fontes públicas oficiais; leitor vê só trecho curto + link; parecer jurídico é **pré-requisito do piloto** | P2, passo 7, seções 7 e 8 |
| W3 Alegações atribuídas | Detecção de atribuição no passo 2, com **dois tipos** (fonte citada × declaração, ver N1/C1) | Passos 2, 4, 6, 7 |
| W4 Pontuação indefinida | Pontuação definida por escrito; caminho "casou" sem número de confiança; calibrador por classe (ver N4/C3) | Passos 5, 6, P3 |
| W5 Mudança de distribuição | **Parcial.** No protótipo: limiar próprio e mais alto para refutada. **Completa só no piloto, com E11 como portão obrigatório** (ver N2/C2) | P3, seções 6 e 7 |
| W6 Mesmo dev para tudo | Train filtrado dividido em **train-A** (limiares) e **train-B** (calibração); dev para escolhas; teste uma vez; Platt no lugar da isotônica | P3, seção 6 |
| W7 Tempo ignorado | Campo `sensivel_ao_tempo`, restrito a alegações sem data explícita (C7); filtro de data na avaliação; checagem antiga não é herdada | Passos 2, 3, 4 |
| W8 Verificador numérico | Incremento; só decide quando entidade, unidade e período coincidem; conflito com NLI vira inconclusiva + revisão | Passo 5 |
| W9 NLI com premissa longa | NLI em janelas de 1 a 2 frases; limiar mais alto para refutada; nota sobre o efeito do máximo entre janelas | Passo 5, P3 |
| W10 Flag `ambigua` ignorada | Consumida no passo 6: limita a saída e manda para revisão | Passos 2, 6 |
| W11 Fluxo humano incompleto | Revisão de refutadas, conflitantes com pessoa, ambíguas e **mudanças de veredito entre versões** (C4); revisão amostral de suportadas como incremento (C5); resultados versionados | Passo 6, seção 8 |
| W12 Confidencialidade | MVP sem chamadas externas com texto não publicado | Passos 4, 7 |
| W13/W14 Passo 7 | MVP só com templates; trecho exibido é a janela decisiva; template da agência mostra a alegação checada (C8) | Passos 3, 7 |
| W15 Latência | Benchmark obrigatório, fila assíncrona e cache de alegações repetidas | Seção 5, E9 |
| W16 Checabilidade sem dados | MVP por regras; BERTimbau como incremento com guia de anotação e kappa; veículo vê quantas frases ficaram de fora | Passo 1 |
| W17 Recalibração e log | Snapshots versionados no log; antes do piloto, o único gatilho de recalibração é a troca de modelo | P3, passo 6 |
| W18 Viés regional | Métricas estratificadas em E11; torna mais urgente a decisão em aberto nº 5 | Seções 7 e 8 |
| W19 Complexidade | MVP enxuto: um só modelo de embedding, regras no passo 1, templates no passo 7 | Todos os passos |
| W20 Inconsistências | Glossário, lista completa de parâmetros, diagrama e tabela-resumo alinhados ao texto (ver N6/C10) | Seções 0.3, 4, 9, 10 |
| W21 Injeção de prompt | Mantido: o LLM não decide e o NLI filtra; teste de red team | Passo 7, E10 |

### 0.2 O que não foi incorporado, e por quê

**Da versão 2 (mantido, com as condições do segundo parecer):**

- **Conjunto de produção (M5):** não é necessário para chegar ao protótipo, mas **é portão obrigatório do piloto** (E11). Sem ele, a precisão de refutada em produção é desconhecida.
- **Calibradores separados para os dois caminhos (M4):** depois do protocolo anti-vazamento, o caminho "casou" quase não aparece no teste. A solução é não exibir confiança numérica nesse caminho, e sim "checada por [agência] em [data]" junto com a alegação checada (C8).
- **Exigir 2 trechos de fontes distintas para refutada (M9):** tornaria refutada muito rara. Entra como ablação (E4).
- **Consultas de busca montadas só com entidades (M12):** ainda revelam o assunto da notícia. A solução é não fazer chamadas externas com texto não publicado no MVP.
- **Decisões de produto de M11:** quem revisa, contestação e ressubmissões ficam com o grupo (seção 8). A revisão amostral de suportadas, que é técnica, foi descrita no passo 6 (C5).
- **Severidade de W2:** alta, não crítica, para um protótipo de pesquisa. Vira crítica em uso real na TAKTA, e por isso o parecer jurídico é pré-requisito do piloto.

**Da versão 3 (correções aplicadas com ajuste ou recusadas em parte):**

- **N5, origem do AVERITEC-BR (C9):** recusada a marcação "a confirmar na EDA" para a origem, porque ela tem fonte. O relatório de datasets v2 do grupo (seção 3.1.1), com base em Delucis et al. (STIL 2025), registra que o AVERITEC-BR é tradução automática do AVeriTeC feita com o Gemma 3. A frase agora cita essa fonte. Os tamanhos dos splits do AVERITEC-BR e a licença continuam "a confirmar na EDA".
- **C9, ressalva de contaminação:** aceita a remoção da ressalva sobre LIAR e FEVER. No lugar, entrou uma ressalva que se aplica ao fluxo: o português dos dados de ajuste dos modelos NLI e do AVERITEC-BR vem de tradução automática.
- **C4, revisão de mudanças de veredito:** aplicada, com uma observação. Uma correção legítima do veículo também muda o veredito, e é exatamente o comportamento que a TAKTA quer estimular. O revisor recebe a diferença entre as versões do texto para distinguir correção de reformulação.
- **C8, checagem antiga:** decidido que ela é registrada no log e mostrada ao veículo e ao revisor, mas **não ao leitor** no MVP. Exibir um veredito possivelmente desatualizado ao lado do resultado atual confundiria mais do que ajudaria.
- **C3, ordem de P3:** aplicada com uma ordem explícita para evitar circularidade. Os limiares do NLI saem do train-A; o calibrador é ajustado no train-B com esses limiares fixos; o limiar de abstenção volta ao train-A, já com o calibrador aplicado.

### 0.3 Glossário

- **Alegação:** afirmação verificável extraída da notícia (passos 1 e 2).
- **Tipo de atribuição** (`tipo_atribuicao`):
  - **`fonte_citada`:** a notícia usa uma fonte como autoridade para um dado ("segundo o IBGE, o desemprego caiu 3%"). O veredito é **do veículo**, e a pergunta é "a fonte diz isso?".
  - **`declaracao`:** a notícia relata a fala de alguém ("o deputado afirmou que Y"). Verifica-se Y, e o resultado é exibido como "declaração de X".
  - **`nenhuma`:** o veículo afirma diretamente.
- **Trecho:** pedaço de cerca de 150 palavras de um documento externo da base P2. Nunca vem da notícia.
- **Janela:** 1 ou 2 frases consecutivas de um trecho, unidade que o NLI lê no passo 5.
- **Janela decisiva:** a janela com a maior probabilidade que sustenta o veredito final. É a que o leitor vê.
- **Trecho forte a favor:** trecho cuja maior `p_confirma` entre as janelas passa de `limiar_suporta`.
- **Trecho forte contra:** trecho cuja maior `p_contradiz` entre as janelas passa de `limiar_refuta`. Por padrão, `limiar_refuta > limiar_suporta`.
- **Conflitante:** uma única classe, com dois motivos possíveis registrados no campo `motivo`: `agencias_divergem` (passo 3) ou `evidencias_divergem` (passo 5).
- **Train-A / train-B:** as duas partes, sem sobreposição, do train filtrado do AVERITEC-BR. A primeira serve aos limiares, a segunda à calibração.
- **Portão do piloto:** condição sem a qual o piloto não começa.
- **Snapshot:** identificador da versão de uma base (P1 ou P2) ou de um conjunto de parâmetros, registrado em cada veredito.
- **Caminho "casou" / "não casou":** se a alegação herdou o veredito de uma checagem (passo 3) ou foi decidida pelo fluxo (passos 4 e 5).

### 0.4 O que mudou na versão 3

| Correção | Problema | Decisão | Onde |
|---|---|---|---|
| C1 | N1, W3 | Campo `tipo_atribuicao` (`fonte_citada`, `declaracao`, `nenhuma`); casos não separáveis viram `declaracao` e são registrados para E6; templates distintos; nova frase na finalidade; fidelidade da citação vira decisão em aberto; E6 ampliado | Finalidade, 0.3, passos 2, 4, 6, 7, seções 7 e 8 |
| C2 | N2, W5 | Item "Validade externa" com cálculo ilustrativo de sensibilidade à taxa-base; aviso de que a precisão do teste não estima a de produção; E11 como portão do piloto, em português e com evidência de P2; correção por prevalência como alternativa a confirmar | Estado, P3, seções 6 e 7 |
| C3 | W6, N4 | Train-A para limiares e abstenção, train-B para calibração, com seed fixa; Platt por classe predita ou com a classe como variável (escolha em E5); abstenção só em classes decisivas; pontuação da inconclusiva só no log | P3, passos 5 e 6, seções 4 e 6 |
| C4 | N3, W11 | Revisão de toda alegação cujo veredito mude de refutada para suportada ou inconclusiva entre versões, com a diferença entre versões mostrada ao revisor | Passo 6 |
| C5 | W11 | Revisão amostral de suportadas descrita no passo 6 como incremento, com taxa a definir pelo grupo | Passo 6, seções 4, 8 e 10 |
| C6 | W2 | Parecer jurídico como pré-requisito do piloto | Seções 7 e 8 |
| C7 | W7 | `sensivel_ao_tempo` só para alegações sem data explícita ou com "atual", "hoje", "agora" | Passo 2 |
| C8 | W13, M4 | Trecho exibido é a janela decisiva; template da agência mostra a alegação checada; checagem antiga vai para o log, o veículo e o revisor, não para o leitor | Passos 3 e 7 |
| C9 | N5 | Fonte da origem do AVERITEC-BR citada; tamanhos dos splits a confirmar na EDA; ressalva de contaminação trocada; "se nenhum erro for observado" em E7 e no passo 2; nota do máximo entre janelas; gatilho de recalibração antes do piloto | Seção 6, P3, passos 2 e 5, E7 |
| C10 | N6, W20 | Critério de `similaridade_minima` descrito; diagrama liga P3 só aos passos 3, 5 e 6; tabela-resumo e parâmetros alinhados ao texto | P3, seções 4, 9 e 10 |

---

# FASE A — PREPARAÇÃO

## P1. Base de checagens

**1. Identificação e o que faz**

- Monta um acervo de checagens publicadas por agências brasileiras, com vereditos convertidos para as 4 classes da FOP e indexados por significado.
- Existe para que o passo 3 reaproveite vereditos humanos antes de o fluxo decidir sozinho.

**2. O que entra**

- **Dados, de fontes externas:** checagens com alegação, veredito textual, agência, data e link, vindas da Google Fact Check Tools API (padrão ClaimReview), das páginas das agências, da página Fato ou Boato e do FactCenter, se o download for obtido.
- **Pré-condições:**
  - tabela de mapeamento "veredito da agência → classe da FOP", produzida na EDA;
  - termos de uso de cada fonte verificados;
  - modelo de embedding único escolhido na EDA.

**3. Como processa**

1. Coletar pela API e pelas páginas.
2. Limpar: remover HTML e normalizar espaços, aspas e acentos (Unicode NFC).
3. Deduplicar: a mesma alegação checada por várias agências vira um grupo com todas as fontes. Se os vereditos divergirem, o grupo recebe "conflitante" com motivo `agencias_divergem`.
4. Mapear cada veredito para uma das 4 classes. Rótulos ambíguos vão para revisão manual.
5. Calcular o embedding de cada alegação.
6. Gravar metadados e vetores, com um identificador de snapshot.
7. Atualizar diariamente de forma incremental. Cada atualização gera um novo snapshot, mas **não** dispara recalibração.

Guarda-se só a alegação curta, os metadados e o link, nunca o artigo inteiro.

**4. Com que ferramentas**

*4.1 Ferramentas*

- `requests`/`httpx` e `trafilatura` para coleta.
- Google Fact Check Tools API.
- **Um único modelo de embedding para P1 e P2:** `intfloat/multilingual-e5-base` ou `BAAI/bge-m3`, escolhido na EDA.
- FAISS para o índice vetorial; SQLite ou PostgreSQL para metadados.

*4.2 Como obter*

- Bibliotecas: `pip install`.
- API: projeto no Google Cloud, ativar a Fact Check Tools API e gerar uma chave.
- Modelos: Hugging Face; conferir licenças. FAISS: licença MIT.

*4.3 Tratamento necessário*

- Chave em variável de ambiente, nunca no código.
- Respeitar robots.txt, intervalo entre requisições e cota da API.
- Normalizar os vetores para comprimento 1.
- Prefixos do e5, se for o escolhido: `query: ` para a alegação nova e `passage: ` para as alegações da base.
- Trocar o modelo de embedding exige reindexar P1 e P2.

*4.4 Alternativas*

- Sem a API: só coleta nas páginas. Sem o FactCenter: só coleta própria.
- pgvector ou Chroma no lugar do FAISS.
- `paraphrase-multilingual-MiniLM-L12-v2` como ablação para o casamento de alegações.

**5. O que sai**

- Tabela `checagens` com `checagem_id`, `alegacao`, `veredito_original`, `classe_fop`, `motivo`, `agencia`, `data`, `url`, `versao_mapeamento`.
- Índice vetorial apontando para `checagem_id`.
- Identificador `snapshot_p1`.
- Entrega: carregados pelo passo 3.

**6. Contribuição para a finalidade**

- É o caminho mais confiável até um veredito, porque reaproveita trabalho humano.
- **Se falhar:** tudo vai para a busca própria, com mais custo e mais erro. Um mapeamento errado converte vereditos humanos de forma errada.

**7. Como verificar**

- Revisão manual de 100 checagens (mapeamento e duplicatas). Com 100 casos, a margem de erro fica em torno de ±10 pontos percentuais, o que basta para erros comuns.

---

## P2. Base de evidência

**1. Identificação e o que faz**

- Monta um índice de trechos curtos de documentos confiáveis.
- Existe porque o passo 4 precisa buscar evidência em algum lugar, e os modelos só leem textos curtos.

**2. O que entra**

- **Produção, no MVP:** conteúdo público de órgãos oficiais (IBGE, INEP, TSE, Diário Oficial, gov.br e portais de dados abertos), conferindo a licença de cada fonte.
- **Produção, depois do parecer jurídico:** texto de veículos e agências. Até lá, deles só entram alegação, metadados e link, como em P1 (W2).
- **Avaliação:** o knowledge store do AVeriTeC, **com o filtro anti-vazamento da seção 6**. Ele está em inglês, e as alegações do AVERITEC-BR estão em português. A EDA decide entre traduzir o knowledge store ou buscar entre línguas. **Se o knowledge store traz data de publicação por documento é algo a confirmar na EDA**; sem essa data, o filtro temporal não é aplicável e isso é registrado como limitação da avaliação.
- **Pré-condição:** lista de fontes aprovada pelo grupo.

**3. Como processa**

1. Coletar os documentos.
2. Extrair o texto principal e normalizar.
3. Remover quase-duplicatas.
4. Cortar em trechos de cerca de 150 palavras, com sobreposição de cerca de 30, sem partir frases.
5. Guardar fonte, domínio, URL e data de publicação de cada trecho.
6. Indexar por palavras (BM25) e por significado (embedding).
7. Registrar um identificador de snapshot.

**4. Com que ferramentas**

*4.1 Ferramentas*

- `trafilatura`, `datasketch` (MinHash), `rank_bm25` (ou OpenSearch em escala), o mesmo modelo de embedding de P1 e FAISS.

*4.2 Como obter*

- `pip install`; OpenSearch via Docker; modelos pelo Hugging Face.

*4.3 Tratamento necessário*

- Prefixo `passage: ` nos trechos, se o modelo for o e5.
- Limite de 512 tokens por trecho.
- Guardar domínio e data de publicação, que são usados pelos filtros.
- **O BM25 não funciona entre línguas.** Na avaliação com o knowledge store em inglês, ou se traduz a consulta ou se usa só a busca por significado. É uma decisão da EDA.

*4.4 Alternativas*

- Busca na web em tempo real (Brave Search API ou Google Programmable Search; a API do Bing foi descontinuada em 2025), **só para texto já publicado** (W12).

**5. O que sai**

- Tabela `trechos` com `trecho_id`, `doc_id`, `texto`, `fonte`, `dominio`, `url`, `data_publicacao`, `posicao`.
- Índices BM25 e vetorial apontando para `trecho_id`.
- Identificador `snapshot_p2`.
- Entrega: carregados pelo passo 4.

**6. Contribuição para a finalidade**

- É a origem de toda evidência que não vem de checagens prontas.
- **Se a base não contiver o fato, o melhor resultado possível é "inconclusiva".** Com só fontes oficiais, espera-se mais inconclusivas, sobretudo em notícias regionais (W18). Isso torna urgente a decisão sobre como exibir "inconclusiva" (seção 8, item 5).

**7. Como verificar**

- Cobertura numa amostra de alegações com resposta conhecida.
- Distribuição das fontes por tema e região.

---

## P3. Limiares e calibração

**1. Identificação e o que faz**

- Escolhe, com dados, os limiares dos passos 3, 5 e 6 (incluindo `similaridade_minima`) e ajusta os calibradores do passo 6.
- Existe porque limiares escolhidos por palpite tornam a abstenção arbitrária.

**2. O que entra**

- **Dados externos, todos com o filtro anti-vazamento da seção 6:**
  - **train** do AVERITEC-BR, dividido em **train-A** (limiares e abstenção) e **train-B** (calibração), sem sobreposição e com seed fixa (`seed_divisao_train`);
  - **dev**, para comparar variantes e ablações;
  - **teste**, usado **uma única vez**, no fim.
- Pares de negação e conjunto de contraste da EDA, para os limiares do passo 3.
- **Pré-condição:** passos 1 a 6 montados com valores padrão. **No MVP, nada em P3 treina modelos**, o que elimina a dependência circular da versão 1.

**3. Como processa**

A ordem importa, para que nenhum parâmetro seja escolhido nos mesmos dados em que é avaliado:

1. **Passo 3:**
   - `similaridade_minima`: escolhida nos pares de paráfrase do conjunto de contraste como um corte de **alta revocação**, isto é, baixo o bastante para não descartar casamentos verdadeiros, porque o NLI filtra depois;
   - `limiar_equivalencia`: escolhido nos pares de negação e no conjunto de contraste.
2. **Passo 5, no train-A:** rodar os passos 4 e 5 e escolher `limiar_suporta` e `limiar_refuta`. O de refutada é escolhido para atingir uma **precisão-alvo** (a definir pelo grupo), porque refutada é a classe mais danosa.
3. **Calibração, no train-B:** com os limiares fixos, ajustar o calibrador com o evento "o veredito final está correto", **por classe predita** (suportada, refutada, conflitante) ou com a classe como variável adicional. A escolha sai de E5.
4. **Abstenção, de volta ao train-A:** com o calibrador aplicado, escolher `limiar_abstencao` pela curva que relaciona quantos casos o fluxo decide e quanto ele erra nesses casos.
5. Reportar tudo com intervalos de confiança por bootstrap.

Se um incremento futuro treinar algum modelo no train do AVERITEC-BR, os limiares passam a ser escolhidos em metade do dev e a calibração na outra metade.

**4. Com que ferramentas**

*4.1 Ferramentas*

- `scikit-learn` (regressão logística para Platt) e código próprio para bootstrap.

*4.2 Como obter*

- `pip install`.

*4.3 Tratamento necessário*

- **Platt ou temperature scaling** enquanto houver poucos milhares de exemplos; a isotônica sobreajusta com poucos dados. A classe conflitante tende a ter poucos exemplos, o que pesa na escolha entre calibrador por classe e calibrador único com a classe como variável (E5).
- **Nota sobre as janelas (W9):** o máximo sobre muitas janelas tende a inflar `p_contradiz` e `p_confirma`. Isso é compensado porque os limiares são ajustados aqui com o mesmo janelamento usado em produção. Se o janelamento mudar, os limiares precisam ser reajustados.
- Seeds fixas.
- **Recalibrar quando:** um modelo for trocado, ou o erro de calibração medido numa amostra rotulada periódica passar do limite. **Antes do piloto, essa amostra não existe, e o único gatilho real é a troca de modelo.** Não recalibrar por atualização diária das bases (W17).

*4.4 Alternativas*

- Validação cruzada no train, se os splits forem pequenos demais.
- **Correção das probabilidades para outra prevalência de classes**, para simular, sem dados novos, cenários em que as alegações falsas são raras. Técnica a confirmar antes do uso; não substitui E11.

**5. O que sai**

- `config.json` com todos os parâmetros da seção 4.
- Calibradores salvos em arquivo, com identificador `versao_config`.
- Entrega: carregados pelos passos 3, 5 e 6.

**6. Contribuição para a finalidade**

- Torna "inconclusiva" uma decisão medida e protege contra refutadas injustas.
- **Se falhar:** o fluxo se abstém demais (inútil) ou de menos (confiante e errado).

**7. Como verificar**

- No teste: F1 por classe, precisão de refutada, erro de calibração e taxa de erro nos casos decididos, todos com intervalos de confiança.
- **A precisão de refutada medida no teste não estima a precisão em produção** (seção 6, item 7). Os limiares de refutada e de abstenção são reajustados no piloto com E11.

---

# FASE B — EXECUÇÃO (a cada notícia)

## 1. Seleção de frases verificáveis

**1. Identificação e o que faz**

- Escolhe, dentro da notícia, as frases com afirmações checáveis.
- Existe porque opiniões e descrições não são verificáveis, e checá-las gera ruído e custo.

**2. O que entra**

- **Dado:** a notícia submetida pelo veículo na TAKTA, `{noticia_id, versao_texto, veiculo_id, titulo, texto, data}`.
- **Pré-condições:** veículo verificado; texto em português (detecção de idioma); texto não vazio.

**3. Como processa**

1. Normalizar o texto e dividir em frases, incluindo o título.
2. **MVP, por regras:** manter frases com pelo menos um destes sinais: número, data, nome próprio (NER) ou verbo de declaração ("afirmou", "segundo"). Verbos de declaração são seguros porque o passo 2 trata a atribuição.
3. Ordenar por número de sinais e manter até `max_alegacoes` (padrão 10).
4. **Registrar quantas frases ficaram de fora do corte**, para exibir ao veículo (W16).

**4. Com que ferramentas**

*4.1 Ferramentas*

- spaCy `pt_core_news_lg` (divisão em frases e NER).
- **Incremento:** BERTimbau (`neuralmind/bert-base-portuguese-cased`) ajustado para checabilidade.

*4.2 Como obter*

- `pip install spacy` e `python -m spacy download pt_core_news_lg`; conferir a licença do modelo português.
- BERTimbau pelo Hugging Face, licença MIT.

*4.3 Tratamento necessário*

- Para o incremento: escrever um guia de anotação, fazer dupla anotação em cerca de 20% das frases e medir a concordância (kappa) antes de treinar. Até onde verificamos, as tarefas de checabilidade do CheckThat! não têm português.

*4.4 Alternativas*

- XLM-R ou mDeBERTa ajustados em dados de outras línguas.

**5. O que sai**

- Itens `{alegacao_id, noticia_id, frase_original, posicao, sinais}` e o total de frases excluídas.
- Entrega: cada item segue separadamente para o passo 2.

**6. Contribuição para a finalidade**

- Define o que será verificado.
- **Se falhar:** alegações importantes nunca são checadas, ou opiniões viram inconclusivas sem sentido.

**7. Como verificar**

- Precisão e revocação num conjunto de frases rotuladas pelo grupo.

---

## 2. Descontextualização e atribuição

**1. Identificação e o que faz**

- Torna cada frase compreensível sozinha, identifica se ela é uma declaração relatada ou um dado atribuído a uma fonte, e marca se ela depende do tempo.
- Existe porque "Ele afirmou que dobrou" não pode ser buscado, porque o veículo não pode ser responsabilizado pela fala que apenas relata, e porque continua responsável pelo dado que atribui a uma fonte (W3, N1).

**2. O que entra**

- **Dados:** itens do passo 1, texto completo, título e data da notícia.
- **Pré-condição:** modelo de NER carregado.

**3. Como processa**

1. **NER** no texto inteiro: pessoas, lugares e organizações com posições.
2. **Substituição de referências**, só quando há **um único** candidato:
   - pronome de pessoa → última pessoa citada;
   - "a prefeitura", "o órgão" → organização mais próxima.
   Havendo dois candidatos, nada é substituído e a alegação recebe `ambigua = true`. Toda substituição feita é registrada.
3. **Detecção e tipo de atribuição**, por padrões:
   - **`fonte_citada`:** "segundo [órgão/instituição/estudo]", "de acordo com [órgão]", "dados do [órgão]". A alegação a verificar é Y, o veredito continua **do veículo**, e a pergunta é "a fonte diz Y?". Guarda-se `fonte_citada = [órgão]`.
   - **`declaracao`:** "[pessoa ou agente político] afirmou/disse/declarou que Y" e citações entre aspas com autor. A alegação a verificar é Y, e `atribuida_a = X`.
   - **`nenhuma`:** sem atribuição.
   - **Casos que as regras não conseguem separar** (ex.: "segundo o ministro"): tratar como `declaracao` e registrar para E6.
4. **Datas relativas** ("ontem", "ano passado") viram datas absolutas a partir da data da notícia.
5. **Sensibilidade ao tempo (C7):** `sensivel_ao_tempo = true` se a alegação tiver número, cargo ou índice **sem data explícita**, ou contiver "atual", "hoje", "agora" e equivalentes. Alegações datadas ("em 2018, o PIB cresceu 1,8%") não são sensíveis ao tempo.
6. Anexar título e frase anterior como contexto.

**4. Com que ferramentas**

*4.1 Ferramentas*

- spaCy `pt_core_news_lg` (NER), `dateparser` e expressões regulares próprias para atribuição.

*4.2 Como obter*

- `pip install`. Os padrões de atribuição são código do grupo.

*4.3 Tratamento necessário*

- As regras são frágeis: discurso indireto sem verbo de declaração escapa, e a fronteira entre fonte citada e declaração nem sempre é clara. Testar em amostra (E6, E7).

*4.4 Alternativas*

- BERTimbau ajustado para NER (corpus HAREM).
- LLM neste passo: resolveria melhor substituição e atribuição, mas quebra a regra de LLM só no final. Fica como meio-termo a testar se as regras falharem muito.

**5. O que sai**

- `{alegacao_id, alegacao, frase_original, contexto, entidades:[{texto, tipo}], substituicoes:[...], ambigua, tipo_atribuicao, atribuida_a, fonte_citada, data_referencia, sensivel_ao_tempo}`.
- Entrega: segue para o passo 3.

**6. Contribuição para a finalidade**

- Alegação autocontida torna a busca possível. O tipo de atribuição define a quem o resultado se refere: ao veículo ou a quem fez a declaração.
- **Se falhar:** busca vazia, veredito sobre a pessoa errada, refutada indevida contra o veículo, ou erro do veículo atribuído à fonte que ele citou.

**7. Como verificar**

- Revisão manual de cerca de 300 alegações com substituição. **Se nenhum erro for observado**, esse tamanho basta para mostrar que a troca de pessoa ocorre em menos de 1% dos casos (E7).
- Teste de atribuição (E6).

---

## 3. Casamento com checagens existentes

**1. Identificação e o que faz**

- Verifica se a alegação já foi checada por uma agência e, se foi, reaproveita o veredito.
- Existe porque um veredito humano é mais confiável e mais barato que um calculado.

**2. O que entra**

- **Dados:** alegação do passo 2 (com `data_referencia` e `sensivel_ao_tempo`) e a base P1.
- **Pré-condições:** mesmo modelo de embedding de P1; `similaridade_minima` e `limiar_equivalencia` do `config.json`.

**3. Como processa**

1. **Cache:** se a mesma alegação normalizada já foi processada com os mesmos snapshots, reaproveitar o resultado.
2. Transformar a alegação em vetor e buscar as `top_k_checagens` mais próximas (padrão 10), descartando as abaixo de `similaridade_minima`.
3. **NLI nos dois sentidos:** só há casamento se a alegação implicar a checagem e a checagem implicar a alegação, ambas acima de `limiar_equivalencia`.
4. **Filtro temporal:** se `sensivel_ao_tempo = true` e a checagem for mais antiga que `idade_max_checagem_meses`, **não herdar**. A alegação segue para o passo 4. A checagem antiga é **registrada no log e mostrada ao veículo e ao revisor, mas não ao leitor** no MVP.
5. Se casar, herdar a `classe_fop`. Se várias checagens casarem com vereditos diferentes, o resultado é "conflitante" com motivo `agencias_divergem`.

**4. Com que ferramentas**

*4.1 Ferramentas*

- Modelo de embedding único + FAISS.
- NLI: `MoritzLaurer/mDeBERTa-v3-base-xnli-multilingual-nli-2mil7` (licença MIT).

*4.2 Como obter*

- Hugging Face.

*4.3 Tratamento necessário*

- Ler no `config` do modelo a ordem dos rótulos (entailment, neutral, contradiction) antes de interpretar as saídas.
- Ativar truncamento.

*4.4 Alternativas*

- `MoritzLaurer/multilingual-MiniLMv2-L6-mnli-xnli`: mais rápido, mas treinado sem português. A escolha sai da EDA.

**5. O que sai**

- **Casou:** `{alegacao_id, origem: "agencia", checagens:[{checagem_id, alegacao_checada, agencia, data, classe_fop, url, prob_equivalencia}], veredito_provisorio, motivo}`. **Pula para o passo 6.**
- **Não casou:** `{alegacao_id, origem: "fop", checagem_antiga_referencia}`. **Segue para o passo 4.**

**6. Contribuição para a finalidade**

- Resolve com segurança os casos já conhecidos.
- **Se falhar por casamento falso:** a alegação herda o veredito de outra afirmação, o erro mais grave deste passo.
- **Se falhar por não achar casamento:** só aumenta o custo.

**7. Como verificar**

- **À parte do teste principal**, que é deduplicado contra P1: taxa de casamentos falsos nos pares de negação e no conjunto de contraste (E2).

---

## 4. Recuperação de evidência

**1. Identificação e o que faz**

- Encontra os trechos externos mais relevantes para a alegação.
- Existe porque, sem evidência, qualquer veredito seria chute.

**2. O que entra**

- **Dados:** alegação, entidades, `tipo_atribuicao`, `fonte_citada`, `data_referencia` e `sensivel_ao_tempo` do passo 2; base P2.
- **Pré-condição:** índices carregados.

**3. Como processa**

Os trechos vêm da base P2, **nunca da notícia**. O processamento é em funil:

1. **Busca por palavras (BM25):** alegação + entidades, `top_k_bm25` trechos (padrão 50).
2. **Busca por significado:** alegação com prefixo `query: `, `top_k_denso` trechos (padrão 50).
3. **Fusão** das duas listas por *Reciprocal Rank Fusion*, somando pontos conforme a posição em cada lista. Resultado: até 100 trechos únicos.
4. **Filtros e ajustes:**
   - remover trechos da própria notícia ou de cópias dela;
   - **na avaliação:** remover domínios de agências de checagem e documentos publicados depois da alegação (seção 6);
   - **em produção:** se `sensivel_ao_tempo = true`, penalizar trechos com data fora de `janela_tempo_meses`;
   - se `tipo_atribuicao = fonte_citada` e a fonte estiver em P2, priorizar trechos do domínio dela, para responder "a fonte diz Y?".
5. **MVP:** ficar com os `top_n_final` primeiros da fusão (padrão 5).
6. **Incremento:** um reranqueador lê os `top_n_rerank` primeiros (padrão 30) em pares (alegação, trecho) e reordena antes do corte final.

**4. Com que ferramentas**

*4.1 Ferramentas*

- `rank_bm25` ou OpenSearch; modelo de embedding único + FAISS.
- **Incremento:** reranqueador `BAAI/bge-reranker-v2-m3`.

*4.2 Como obter*

- `pip install`; modelos pelo Hugging Face, conferindo licenças.

*4.3 Tratamento necessário*

- A consulta passa pela mesma limpeza da indexação.
- O reranqueador tem limite de 512 tokens e é o componente mais lento do fluxo: medir latência (E9).
- **Confidencialidade (W12):** nenhuma consulta com texto não publicado sai para serviços externos no MVP.

*4.4 Alternativas*

- Cross-encoders multilíngues treinados no mMARCO no lugar do bge-reranker.
- Busca na web só para textos já publicados.

**5. O que sai**

- `{alegacao_id, trechos:[{trecho_id, texto, fonte, dominio, url, data, nota}]}`, com 0 a 5 trechos.
- Entrega: segue para o passo 5.

**6. Contribuição para a finalidade**

- Fornece o material em que o veredito se apoia.
- **Se falhar:** sem trecho relevante, o resultado vira inconclusiva; com trecho errado, o veredito sai errado.

**7. Como verificar**

- No AVERITEC-BR filtrado: revocação@5 (a evidência-ouro aparece entre os 5?) e MRR, com ablações do funil (E3).

---

## 5. Veredito

**1. Identificação e o que faz**

- Compara a alegação com os trechos e decide entre as 4 classes.
- Existe porque é aqui que a FOP decide, com componentes auditáveis e sem LLM.

**2. O que entra**

- **Dados:** a alegação (ou o conteúdo Y, se houver atribuição) e os trechos do passo 4.
- **Pré-condição:** `limiar_suporta` e `limiar_refuta` definidos em P3.

**3. Como processa**

1. **Dividir cada trecho em janelas** de 1 a 2 frases (W9).
2. **NLI por janela:** a janela é a premissa, a alegação é a hipótese. Saem três probabilidades: confirma, contradiz, não diz nada.
3. **Por trecho,** guardar a maior `p_confirma` e a maior `p_contradiz` entre as janelas, e qual janela as produziu.
4. **Classificar os trechos:** forte a favor se a maior `p_confirma` ≥ `limiar_suporta`; forte contra se a maior `p_contradiz` ≥ `limiar_refuta`.
5. **Agregar por regras:**

| Situação | Veredito provisório | Pontuação |
|---|---|---|
| Algum forte a favor e nenhum forte contra | Suportada | maior `p_confirma` entre os fortes |
| Algum forte contra e nenhum forte a favor | Refutada | maior `p_contradiz` entre os fortes |
| Fortes nos dois sentidos | Conflitante (`evidencias_divergem`) | menor entre a maior `p_confirma` e a maior `p_contradiz` |
| Nenhum forte | Inconclusiva | sem pontuação calibrada (ver abaixo) |

   Para a inconclusiva, registra-se no log apenas a maior probabilidade decisiva observada, para diagnosticar casos que ficaram perto do limiar. Ela não é calibrada nem exibida, porque aplicar abstenção a uma inconclusiva seria circular (N4).

6. **Incremento, verificador numérico** (W8):
   - extrai quantidades, unidades e relações ("dobrou" = ×2, "metade", "%", "p.p.", "mil", "mi", "bi", "R$");
   - **só decide** quando entidade, unidade e período coincidem entre alegação e trecho, com tolerância `tolerancia_numerica` ajustada em P3;
   - se discordar do NLI, o resultado vira **inconclusiva** e vai para revisão.

**4. Com que ferramentas**

*4.1 Ferramentas*

- O mesmo NLI do passo 3.
- Incremento: código próprio para números.

*4.2 Como obter*

- NLI pelo Hugging Face. Verificador numérico: não se aplica, porque é código do grupo.

*4.3 Tratamento necessário*

- Tokenizar com a premissa primeiro e `truncation="only_first"`, para que o truncamento nunca corte a alegação.
- Conferir a ordem dos rótulos do modelo.
- Normalizar números no padrão brasileiro: "8.200" é oito mil e duzentos, "8,2" é decimal.
- O máximo entre muitas janelas infla as probabilidades; os limiares de P3 compensam isso desde que o janelamento não mude (ver P3 §4.3).

*4.4 Alternativas*

- No lugar das regras, um classificador ajustado sobre (alegação, 5 trechos), no formato AVERITEC-BR. Se for adotado, os limiares de P3 passam para o dev.

**5. O que sai**

- `{alegacao_id, veredito_provisorio, motivo, pontuacao, janela_decisiva, evidencias:[{trecho_id, max_p_confirma, max_p_contradiz, papel}], numerico:{aplicado, resultado}}`.
- Entrega: segue para o passo 6.

**6. Contribuição para a finalidade**

- É a decisão, apoiada em trechos identificáveis.
- **Se falhar:** o veredito sai errado. As métricas por classe mostram onde.

**7. Como verificar**

- No teste filtrado: F1 por classe, precisão de refutada e confusão entre refutada e inconclusiva, com ablações (E4) e o conjunto de contraste (E2).

---

## 6. Calibração, abstenção e revisão

**1. Identificação e o que faz**

- Transforma a pontuação em confiança, decide quando se abster, encaminha casos sensíveis para revisão humana e registra tudo.
- Existe para que "inconclusiva" não seja lida como "suspeita" e para que nenhuma decisão danosa produza efeito sem um humano.

**2. O que entra**

- **Dados:** a saída do passo 3 (origem `agencia`) ou do passo 5 (origem `fop`), mais as flags do passo 2 (`ambigua`, `tipo_atribuicao`, `atribuida_a`, `fonte_citada`) e os resultados de versões anteriores da mesma notícia, se houver.
- **Pré-condição:** calibradores e limiar de abstenção de P3.

**3. Como processa**

1. **Origem `agencia`:** não há confiança numérica. O resultado é exibido como "checada por [agência] em [data]", junto com a alegação checada.
2. **Origem `fop`:**
   - **suportada, refutada e conflitante:** a pontuação passa pelo calibrador da classe (ou pelo calibrador com a classe como variável) e vira confiança. Abaixo de `limiar_abstencao`, o veredito vira **inconclusiva** com motivo "baixa confiança".
   - **inconclusiva:** não recebe confiança numérica.
3. **Encaminhar para revisão humana:**
   - toda **refutada**;
   - toda **conflitante** envolvendo pessoa (entidade do tipo pessoa);
   - toda alegação com `ambigua = true`, cuja saída fica limitada a inconclusiva até a revisão;
   - todo conflito entre o verificador numérico e o NLI;
   - **toda alegação cujo veredito mude de refutada para suportada ou inconclusiva entre versões da mesma notícia (C4).** O revisor recebe a diferença entre as versões do texto: uma correção legítima do veículo é o comportamento desejado, e o que se quer barrar é a reformulação que muda só a forma;
   - **incremento (C5):** uma amostra aleatória das **suportadas**, com taxa `taxa_revisao_suportadas` a definir pelo grupo, para estimar quantas suportadas estão erradas.
4. **Registrar no log de auditoria:** `noticia_id`, `versao_texto`, `snapshot_p1`, `snapshot_p2`, versões dos modelos, `versao_config`, substituições feitas, tipo de atribuição e o resultado. Ressubmissões da mesma notícia ficam ligadas entre si, e o histórico é mostrado ao revisor.

**4. Com que ferramentas**

*4.1 Ferramentas*

- `scikit-learn` (calibradores de P3) e um banco de dados para o log e a fila de revisão.

*4.2 Como obter*

- `pip install`.

*4.3 Tratamento necessário*

- Monitorar a taxa de abstenção e o tamanho da fila de revisão.

*4.4 Alternativas*

- Sem calibrador: limiar direto sobre a pontuação, menos confiável.

**5. O que sai**

- `{alegacao_id, origem, veredito_final, confianca (só origem fop e classes decisivas), status: final|pendente_revisao, motivo, tipo_atribuicao, atribuida_a, fonte_citada, evidencias}`.
- Entrega: segue para o passo 7; os casos marcados entram na fila de revisão. Quem revisa e se a publicação espera a revisão são decisões em aberto (seção 8).

**6. Contribuição para a finalidade**

- Materializa o princípio de declarar incerteza em vez de chutar, e mantém o humano nas decisões danosas.
- **Se falhar:** o fluxo aparenta mais certeza do que tem, ou pune uma notícia sem revisão.

**7. Como verificar**

- Erro de calibração e diagrama de confiabilidade no teste, por classe.
- Taxa de erro nos casos decididos em comparação com a taxa de abstenção.
- Volume esperado da fila de revisão.

---

## 7. Explicação

**1. Identificação e o que faz**

- Explica, em linguagem simples, por que cada alegação recebeu seu veredito, citando as fontes.
- Existe porque o leitor precisa entender o porquê para julgar por conta própria.

**2. O que entra**

- **Dados:** a saída do passo 6 com os trechos (texto, fonte, data, link) e a janela decisiva.
- **Pré-condição:** templates definidos. No incremento, LLM disponível.

**3. Como processa**

**MVP, só templates (W13, W14, W19):**

- **Origem `agencia`:** "Checada por [agência] em [data]: [classe]. Alegação checada: '[alegacao_checada]'. [link]". Mostrar a alegação checada permite ao leitor comparar com a da notícia e julgar o casamento (C8).
- **Origem `fop`:** "Segundo [fonte] ([data]): '[janela decisiva]'. [frase de conclusão da classe]. [link]". O texto exibido é a **janela decisiva**, cortada em `max_palavras_trecho_exibido` (padrão 25) (W2, C8).
- **`tipo_atribuicao = declaracao`:** o texto começa com "Declaração de [X]:".
- **`tipo_atribuicao = fonte_citada`:** o texto começa com "A notícia atribui este dado a [fonte]:". O resultado continua sendo da notícia.
- **Inconclusiva sem trechos:** "Não encontramos evidência suficiente para verificar esta alegação."

**Incremento, com LLM:**

1. Um prompt fixo com o veredito e os trechos entre delimitadores pede **só frases factuais** com citações [T1], [T2]..., sem alterar o veredito.
2. O NLI confere cada frase factual contra o trecho que ela cita; frase sem sustentação é cortada.
3. **A frase de conclusão e o prefixo de atribuição continuam vindo do template.**
4. Se não sobrar frase válida, volta o template do MVP.

**4. Com que ferramentas**

*4.1 Ferramentas*

- MVP: código próprio de templates.
- Incremento: LLM aberto de cerca de 7 a 8B de parâmetros com bom português, rodando localmente via Ollama ou llama.cpp; o mesmo NLI dos passos 3 e 5.

*4.2 Como obter*

- Templates: não se aplica, porque são código do grupo.
- LLM: instalar o Ollama e baixar o modelo, conferindo a licença.

*4.3 Tratamento necessário*

- Temperatura baixa (0 a 0,3) e limite de tokens.
- **Os trechos são texto externo e podem conter instruções maliciosas:** vão entre delimitadores, e o prompt manda tratá-los como dados. O dano possível é limitado, porque o LLM não decide o veredito e o NLI filtra as frases.
- **LLM por API externa só com retenção zero contratual**, porque o texto ainda não foi publicado (W12).

*4.4 Alternativas*

- Templates sempre, que é o próprio MVP.

**5. O que sai**

- **Por alegação:** `{alegacao_id, frase_original, tipo_atribuicao, atribuida_a, fonte_citada, veredito_final, origem, confianca, status, explicacao, fontes:[{url, fonte, data}]}`.
- **Por notícia:** `{noticia_id, versao_texto, alegacoes:[...], frases_excluidas}`.
- **Entrega à TAKTA:** primeiro ao veículo, antes da publicação; depois, junto da notícia, ao leitor.

**6. Contribuição para a finalidade**

- Transforma a decisão técnica em informação que o leitor consegue avaliar, dizendo a quem cada resultado se refere.
- **Se falhar:** veredito sem explicação, contrariando a proposta da TAKTA. O template evita a falha total.

**7. Como verificar**

- Incremento: taxa de frases cortadas pelo NLI e teste cego LLM × template com leitores, em cerca de 50 explicações (E8).

---

# Seções de apoio

## 4. Parâmetros do `config.json`

Todos os valores são padrões iniciais; os marcados com (P3) são ajustados com dados, e os marcados com (grupo) são decisões do grupo.

| Parâmetro | Passo | Padrão |
|---|---|---|
| `max_alegacoes` | 1 | 10 |
| `top_k_checagens` | 3 | 10 |
| `similaridade_minima` | 3 | (P3, corte de alta revocação) |
| `limiar_equivalencia` | 3 | (P3) |
| `idade_max_checagem_meses` | 3 | 12 (grupo) |
| `top_k_bm25`, `top_k_denso` | 4 | 50, 50 |
| `top_n_rerank` | 4 (incremento) | 30 |
| `top_n_final` | 4 | 5 |
| `janela_tempo_meses` | 4 | (grupo) |
| `limiar_suporta` | 5 | (P3, train-A) |
| `limiar_refuta` | 5 | (P3, train-A, pela precisão-alvo; reajuste no piloto com E11) |
| `tolerancia_numerica` | 5 (incremento) | (P3) |
| `seed_divisao_train` | P3 | valor fixo registrado |
| `proporcao_train_a` | P3 | 0,5 (padrão inicial) |
| `limiar_abstencao` | 6 | (P3, train-A; reajuste no piloto com E11) |
| `taxa_revisao_suportadas` | 6 (incremento) | (grupo) |
| `max_palavras_trecho_exibido` | 7 | 25 |

## 5. Operação

- **Fila assíncrona:** a notícia entra numa fila e o veículo é avisado quando o resultado fica pronto.
- **Cache** de alegações repetidas (passo 3), invalidado quando mudam os snapshots.
- **Benchmark obrigatório** de tempo e custo por notícia com 10 alegações, em CPU e GPU, antes de fixar `top_n_rerank` e o número de janelas (E9).

## 6. Protocolo de avaliação (anti-vazamento)

1. **Splits do AVERITEC-BR.** Segundo o relatório de datasets v2 do grupo (seção 3.1.1, com base em Delucis et al., STIL 2025), o AVERITEC-BR é uma tradução automática do AVeriTeC feita com o Gemma 3. O AVeriTeC original tem 3.068 / 500 / 1.000 alegações em train / dev / teste (relatório, seção 3.1.2). **Os tamanhos dos splits do AVERITEC-BR, a disponibilidade dos rótulos do teste e a licença são a confirmar na EDA.** Uso: train dividido em train-A (limiares e abstenção) e train-B (calibração); dev para comparar variantes; teste uma única vez, no fim.
2. **Base de evidência na avaliação:**
   - remover domínios de agências de checagem;
   - remover documentos publicados depois da data da alegação. **A presença de data por documento no knowledge store é a confirmar na EDA**; sem ela, este filtro não é aplicável e a limitação é registrada junto dos resultados.
   Em produção, checagens são evidência legítima; na avaliação, entregariam a resposta.
3. **Deduplicar o teste contra P1** (MinHash + embedding). Com isso, o passo 3 praticamente não atua no teste, e por isso é **avaliado à parte** (E2).
4. **Reportar métricas com e sem o passo 3** e com intervalos de confiança por bootstrap.
5. **Métricas principais:** F1 por classe, precisão de refutada, confusão refutada × inconclusiva, erro de calibração por classe e taxa de erro nos casos decididos em função da abstenção.
6. **Ressalva sobre o português dos dados:** os modelos NLI foram ajustados com pares em que o português vem de tradução automática, e o AVERITEC-BR também é traduzido. O desempenho sobre português jornalístico real só é medido em E11.
7. **Validade externa.** O sistema avaliado não é o sistema de produção: na avaliação, as alegações são traduzidas e a evidência vem de um knowledge store em inglês; em produção, o texto é jornalismo brasileiro e a evidência vem de fontes oficiais brasileiras. Além disso, a proporção de alegações falsas muda. **Ilustração com números hipotéticos:** se 5% das alegações em produção forem falsas, e o sistema refutar 60% das falsas e 5% das verdadeiras, a precisão de refutada será

   0,05 × 0,6 ÷ (0,05 × 0,6 + 0,95 × 0,05) ≈ **39%**,

   mesmo que no AVeriTeC ela chegue a 90%. Por isso, **a precisão de refutada medida no teste não estima a precisão em produção**, e os limiares de refutada e de abstenção são reajustados no piloto com E11.

## 7. Experimentos priorizados

**Necessários para o protótipo (MVP):**

- **E1. Auditoria de vazamento:** fração do teste com casamento em P1 e com a checagem-fonte entre os trechos; métricas com e sem o filtro.
- **E2. Conjunto de contraste:** cerca de 100 perturbações controladas ("dobrou / não dobrou / triplicou", "8.200 / 8,2", "2% / 2 p.p.", "ontem / ano passado", troca de pessoa), mais os pares de negação e de paráfrase da EDA. Mede a taxa de inversão correta, os casamentos falsos do passo 3 e serve para escolher `similaridade_minima`.
- **E3. Ablações do passo 4:** BM25, denso, fusão, e a escolha entre e5 e bge-m3, medidos por revocação@5 e MRR.
- **E4. Ablações do passo 5:** trecho inteiro × janelas; 1 × 2 trechos fortes para refutada.
- **E5. Calibração:** Platt por classe predita × Platt com a classe como variável × temperature scaling, com diagrama de confiabilidade por classe e intervalos por bootstrap, sempre no train-B.
- **E6. Atribuição:** três grupos de pares, medindo a classificação de `tipo_atribuicao` e as refutadas atribuídas à pessoa ou ao veículo errado:
  - "X afirmou que Y" (Y falso) × "Y" dito pelo veículo;
  - dado atribuído a uma fonte **corretamente**;
  - dado atribuído a uma fonte **incorretamente** (a fonte diz outro valor).
  Inclui os casos que as regras não conseguiram separar.

**Portões do piloto (o piloto não começa sem eles):**

- **E11. Conjunto de produção:** 200 a 300 alegações em português de notícias reais, incluindo veículos regionais, com evidência buscada nas fontes oficiais de P2 e rotuladas pelo grupo. Mede falsos alarmes de refutada e taxa de abstenção, **reajusta `limiar_refuta` e `limiar_abstencao`** e estratifica as métricas por região, tema e porte do veículo (W18).
- **Parecer jurídico** sobre direito autoral, reuso de vereditos de agências e LGPD (seção 8, item 7), porque o piloto é uso real na TAKTA (W2).

**Com os incrementos:**

- **E7. Descontextualização:** revisão de cerca de 300 alegações com substituição, comparando com a opção sem substituição. Se nenhum erro for observado, esse tamanho mostra troca de pessoa abaixo de 1%.
- **E8. LLM × template:** teste cego de clareza e fidelidade.
- **E9. Carga:** tempo e custo por notícia e comportamento num pico de 10 vezes o volume médio.
- **E10. Red team:** injeção de instruções nos trechos e simulação de reformulação até obter "suportada", confirmando que C4 a leva à revisão.

## 8. Decisões em aberto (do grupo)

1. Quem revisa os casos da fila: moderação da TAKTA, o veículo ou ambos?
2. A notícia pode ser publicada com revisão pendente? O que o leitor vê enquanto isso?
3. Haverá canal de contestação do veículo? Com que prazo, e a divergência fica visível ao leitor?
4. Quantas ressubmissões são permitidas, e como o histórico é exibido?
5. **(Urgente)** Como exibir "inconclusiva" para que não seja lida como "suspeita"? Pede teste com usuários. A urgência vem de que, com só fontes oficiais em P2, as notícias regionais tendem a receber mais inconclusivas (W18).
6. Qual é a precisão-alvo para refutada no protótipo, e qual será exigida no piloto?
7. **(Pré-requisito do piloto)** Base legal (LGPD) para processar alegações sobre pessoas identificadas, e viabilidade de reusar vereditos de agências e texto de veículos. Pede parecer jurídico.
8. Índice próprio ou busca na web para textos já publicados? Decidir com E3, custo e o parecer do item 7.
9. Meta de latência entre a submissão e o resultado, e hardware disponível.
10. Política de reverificação: um veredito continua exibido quando surge evidência nova?
11. **A FOP deve verificar também a fidelidade da citação** ("X disse mesmo isso?") nas alegações do tipo `declaracao`? Exibir "declaração de [pessoa]: refutada" sobre uma citação imprecisa tem risco reputacional e jurídico para a pessoa citada.
12. Qual taxa de revisão amostral de suportadas (`taxa_revisao_suportadas`) a fila comporta?

## 9. Diagrama da cadeia

```
PREPARAÇÃO
 P1 Checagens das agências → base de checagens + índice (snapshot_p1) ······ usada em [3]
 P2 Fontes oficiais        → trechos + índices BM25 e vetorial (snapshot_p2) · usada em [4]
 P3 Train-A / Train-B      → limiares + calibradores (versao_config) ········ usados em [3], [5], [6]

EXECUÇÃO
 Notícia submetida
   → [1] frases verificáveis (regras)
   → [2] alegações autocontidas + tipo de atribuição + tempo
   → [3] já checada e ainda válida?                ← P1 + limiares de P3
          ├─ sim (origem agência) ───────────────────────┐
          └─ não → [4] trechos de evidência        ← P2   │
                 → [5] NLI em janelas + regras     ← P3   │
   → [6] calibração, abstenção, revisão, log  ← P3 ←─────┘
   → [7] explicação (templates; LLM como incremento)
   → Resultado por alegação, entregue ao veículo antes da publicação
   → FINALIDADE: veículo corrige, leitor avalia por conta própria
```

## 10. Tabela-resumo

| Passo | MVP | Incremento | Entrada | Saída | Contribuição |
|---|---|---|---|---|---|
| P1 Base de checagens | Fact Check Tools API, embedding único, FAISS | MiniLM-L12 como ablação | Checagens das agências | Base indexada + snapshot | Reaproveita vereditos humanos |
| P2 Base de evidência | Fontes oficiais, BM25, embedding único | Veículos e agências após parecer | Documentos | Trechos indexados + snapshot | Origem da evidência |
| P3 Limiares e calibração | Limiares no train-A; Platt por classe no train-B | Reajuste com E11 (portão do piloto) | Train-A/B e dev do AVERITEC-BR, contraste | `config.json` + calibradores | Decisões medidas |
| 1 Frases verificáveis | Regras + spaCy | BERTimbau ajustado | Notícia | Frases checáveis + nº excluídas | Define o que verificar |
| 2 Descontextualização e atribuição | NER, regras, tipo de atribuição, tempo | LLM se as regras falharem | Frases + texto | Alegações autocontidas com flags | Busca possível; define a quem o resultado se refere |
| 3 Casamento | Embedding + NLI bidirecional + filtro temporal | — | Alegação + P1 | Veredito da agência ou "não casou" | Atalho confiável |
| 4 Recuperação | BM25 + denso + fusão + filtros | Reranqueador | Alegação + P2 | Até 5 trechos | Material do veredito |
| 5 Veredito | NLI em janelas + regras | Verificador numérico | Alegação + trechos | Veredito + pontuação + janela decisiva | A decisão, auditável |
| 6 Calibração e revisão | Calibrador por classe, abstenção, fila (inclui mudanças de veredito), log versionado | Revisão amostral de suportadas | Veredito provisório | Veredito final + status | Incerteza honesta, humano no dano |
| 7 Explicação | Templates por origem e tipo de atribuição | LLM local + NLI | Veredito + trechos | Resultado por alegação | Leitor entende o porquê e a quem se refere |
