# Anexo Técnico (não-normativo): como construir a FOP

- **Slug**: takta
- **Created**: 2026-09-22
- **Status**: insumo para a decisão do estágio `shape`; **não** é o artefato de decisão.
- **Propósito**: atender ao pedido do grupo de que as sugestões de arquitetura venham acompanhadas de um plano sólido e das ferramentas. O detalhamento definitivo (spec/plan/tasks) pertence ao `/speckit.specify` em diante.
- **Nota (2026-09-27)**: superado pelo `fluxo-ia-fop-v3.md` (plano definitivo): veredito por **NLI em janelas + regras** (não encoder ajustado), extração por regras+spaCy, casamento com checagens, calibração/abstenção, revisão humana e explicação por templates. Mantido como registro histórico.

---

## 1. Arquitetura recomendada em 3 camadas

```
Notícia (texto)
   │
   ▼
[1] Extração & normalização de alegações   ← generativo, saída estruturada
   │   (claims autocontidas e verificáveis)
   ▼
[2] Recuperação de evidência               ← não generativo
   │   BM25 + busca densa + reranking sobre base de evidência
   ▼
[3] Veredito em 4 classes                  ← encoder PT-BR ajustado
   │   suportada | refutada | inconclusiva | evidências conflitantes
   ▼
[4] Explicação ancorada + interface web    ← generativo grounded + UI
```

Princípio de ouro: **o veredito nunca é do LLM**. O generativo atua nas pontas (extrair, explicar); o núcleo de confiabilidade é recuperação + classificador supervisionado.

---

## 2. Camada 1 — Extração e normalização de alegações

**Objetivo**: transformar a notícia em afirmações autocontidas e verificáveis, removendo opinião e material acessório (tarefa do CheckThat! 2025 Task 2).

- **Ferramentas**:
  - Segmentação: `spaCy` (`pt_core_news_lg`) para sentenças/parágrafos.
  - Extração/normalização: LLM com **saída estruturada** (JSON schema) — via API (OpenAI/Anthropic/Gemini) ou local (`Ollama` + `qwen2.5:7b-instruct`, bom em português). Constrained decoding: `outlines` ou `guidance` se local.
  - Dados de referência: fatia PT-BR do **CheckThat! 2025 Task 2** (2.183 instâncias) para ajuste/avaliação; `transformers` + `datasets`.
- **Baseline sem generativo**: regras de segmentação + filtro de sentenças declarativas (para a Opção A / degradação).

## 3. Camada 2 — Recuperação de evidência

**Objetivo**: buscar externamente o material probatório para cada alegação.

- **Bases de evidência**:
  - `FactCenter` (11.647 checagens de 6 agências BR): alegação + veredito + justificativa → base PT-BR primária.
  - Repositório de evidência do **AVeriTeC** (~1.000 artigos web por alegação; licença CC BY-NC 4.0) para transferência.
  - Opcional: Wikipédia PT via API para evidência factual estruturada.
- **Ferramentas de recuperação**:
  - Léxica: `rank_bm25` (simples) ou **Pyserini/Lucene** (robusto).
  - Densa: `sentence-transformers` + **FAISS**; encoder multilíngue forte: `BAAI/bge-m3` ou `intfloat/multilingual-e5-large` (ambos cobrem PT). Alternativa nativa: BERTimbau em `sentence-transformers`.
  - Reranking: cross-encoder `BAAI/bge-reranker-v2-m3` (multilíngue).
- **Saída**: top-k trechos de evidência com fonte, data e URL.

### 3.1 Como funciona a recuperação (busca esparsa, densa e reranker)

Recuperar evidência é um problema clássico de **recuperação de informação (IR)**: dada uma alegação, encontrar em milhares de documentos os trechos que a provam ou a desmentem. Passar tudo para o classificador é inviável (lento e ruidoso), então usa-se um funil:

```
alegação → [recuperação: rápida e abrangente] → ~100 candidatos
         → [reranker: lento e preciso]         → ~5 melhores evidências
         → [classificador] → veredito
```

**Ponto crítico:** se a evidência certa não entrar no funil, o classificador não tem como acertar. O gargalo é a recuperação — por isso o relatório do Grupo 5 conclui que o ganho vem da recuperação, não do tamanho do modelo.

**Busca esparsa (léxica) — BM25.** Casa *palavras*. Pontua cada documento por frequência do termo (com saturação: a 10ª ocorrência vale pouco mais que a 5ª), raridade do termo (IDF: "Bolsa Família" vale muito; "de"/"que" quase nada) e normalização por comprimento. *Forças*: instantânea, determinística, ótima com nomes, siglas, datas e números — justamente o que importa em fact-checking. *Fraqueza*: não entende significado; não liga "auxílio ampliado" a "benefício reajustado". Libs: `rank_bm25`, `Pyserini`.

**Busca densa (semântica) — bi-encoder + embeddings.** Casa *significado*. Um encoder converte cada trecho em um vetor (ex.: ~768 números) e a alegação em outro; a relevância é a similaridade de cosseno. Chama-se **bi-encoder** porque alegação e trecho passam por *lados separados* do modelo — os vetores da base são pré-computados **uma vez** e guardados; em produção só se vetoriza a alegação e busca-se o vizinho mais próximo num índice **ANN** (`FAISS`). *Forças*: capta paráfrase/sinônimos e funciona cross-lingual (evidência em inglês, alegação em PT). *Fraqueza*: "burra" com detalhes exatos — "aumentou 50%" e "aumentou 5%" ficam quase no mesmo lugar; negações passam batido. Modelos: `BAAI/bge-m3`, `intfloat/multilingual-e5-large`; nativo PT: BERTimbau via `sentence-transformers`.

**Híbrida.** Roda BM25 (segura números/nomes) **e** a busca densa (segura paráfrases) e funde os rankings — via **RRF** (Reciprocal Rank Fusion) ou soma de scores normalizados. É a configuração recomendada: cada método cobre a fraqueza do outro.

**Reranker — precisão sobre poucos.** Depois de trazer ~100 candidatos, um **cross-encoder** escolhe os ~5 melhores. Diferença essencial do bi-encoder:

| | Bi-encoder (busca densa) | Cross-encoder (reranker) |
|---|---|---|
| Processa | alegação e trecho **separados** | alegação + trecho **juntos**, no mesmo forward |
| Atenção | não cruza os dois | **atenção cruzada** palavra a palavra |
| Velocidade | rapidíssimo, pré-computável | lento, não pré-computável |
| Precisão | boa para filtrar muito | **muito mais precisa** |

Como o cross-encoder é caro, roda só em poucas dezenas/centenas de candidatos — daí o funil. Ferramenta: `BAAI/bge-reranker-v2-m3` (multilíngue). O reranking também ajuda a detectar **evidências conflitantes**: dois trechos que dizem coisas opostas sobem juntos, sinalizando a classe correspondente.

**Exemplo completo.** Alegação: *"O Bolsa Família foi aumentado em 50% em 2026."*
1. **BM25** traz checagens com "Bolsa Família" e "2026".
2. **Densa** traz checagens sobre "reajuste do benefício social" sem as palavras exatas.
3. **Fusão** → ~100 trechos candidatos.
4. **Reranker** → top 5: (a) "reajuste de 8% em 2026" (contradiz o valor); (b) "benefício ampliado" (relacionado); (c) "aumento de 50%..." (reforça).
5. **Classificador** lê alegação + top 5 → como (a) contradiz e (c) apoia, tende a **evidências conflitantes** ou **refutada**.
6. **Explicação** cita (a) e (c) com fonte e data.

Se o passo 1–2 não trouxesse o trecho "reajuste de 8%", o sistema poderia devolver "inconclusiva" — um erro **de recuperação**, não de classificação. Mede-se por **Recall@k**: a evidência correta estava entre os k recuperados? (gate da Fase F1).

**Armadilhas no caso da FOP:** *chunking* (tamanho dos trechos: grande = ruído, pequeno = perde contexto); *vazamento temporal* (não recuperar evidência posterior à notícia); *idioma* (AVeriTeC em inglês → priorizar FactCenter/Wikipédia PT e encoder multilíngue); *neutralidade* (o ranking pode favorecer a evidência que concorda — é preciso recuperar **evidência contrária também**, senão a classe "refutada" fica cega).

### 3.2 Limite temporal da verificação (a pergunta "e se a base parar em 2025?")

**Sim, existe uma dependência temporal — mas o comportamento correto não é "falso", é "inconclusiva".**

Um sistema de verificação por evidência só consegue checar uma alegação se **existir evidência** sobre ela no momento da consulta. Se avaliarmos em 2027 uma alegação sobre 2027 e a base de evidência estiver **congelada em 2025**, simplesmente não há material probatório: a saída honesta (e a prevista no escopo da FOP) é **inconclusiva / não há evidência suficiente** — jamais "falsa". Isso é uma limitação, mas também é exatamente o princípio que protege o usuário: ausência de prova não é prova de falsidade.

Distinções importantes:

- **Base congelada (dataset) vs. base viva (sistema).** Nos datasets usados para *treinar e avaliar* (AVERITEC-BR etc.), a base é congelada — por isso o protocolo exige **partição temporal de teste**. Num sistema real em operação, a camada de evidência deve ser **continuamente atualizada** (ingestão de notícias por RSS, APIs de agências de checagem, Wikipédia, dados oficiais). Assim, uma notícia de 2027 pode ser checada contra fontes de 2027 coletadas **até o instante da publicação**.
- **A FOP atua "on publication".** Por definição, ela usa a evidência **disponível naquele momento**; evidência posterior é proibida (vazamento temporal). Consequência: para um fato **em desenvolvimento**, a FOP legitimamente dirá "inconclusiva" até acumular evidência. Isso é o sistema funcionando, não falhando.
- **O que não é verificável em princípio** (não é limitação de base): previsões/futuro, opiniões, alegações vagas e fatos privados sem registro público.
- **Implicação para o MVP da residência:** uma base estática basta para **avaliação offline** (medir acurácia contra o AVERITEC-BR). Para a **demo**, é recomendável deixar explícito (a) a cobertura temporal da base e (b) o conceito de atualização contínua, exibindo "inconclusiva" quando a evidência não estiver disponível. Uma fonte viva adequada é o **Fato ou Boato (TSE)**, com filtros por período.

## 4. Camada 3 — Veredito em 4 classes

**Objetivo**: classificar a alegação dadas a alegação + evidência recuperada, no esquema exato do AVeriTeC.

- **Modelo**: encoder PT-BR ajustado; entrada = alegação + evidência recuperada (concatenada/formatada com separadores). Candidatos:
  - `neuralmind/bert-base-portuguese-cased` (BERTimbau base) — recomendado como ponto de partida.
  - `neuralmind/bert-large-portuguese-cased` (se houver GPU).
  - Alternativas multilíngues: `xlm-roberta-base`, `microsoft/mdeberta-v3-base`.
- **Treino**: `transformers` `Trainer` + `datasets`; `peft`/LoRA se o recurso for apertado; `accelerate`; `scikit-learn` para métricas.
- **Dados**: ajuste no **AVERITEC-BR** (4.568 alegações, 4 classes, evidência QA, kappa 0,619); transferência do AVeriTeC original. Classe de **evidências conflitantes** é rara — aumentar com o desenho do **Emergent** (postura de múltiplos documentos).
- **Baselines obrigatórios**: (a) classe majoritária; (b) BM25 + regressão logística; (c) só texto da alegação (sem evidência) para demonstrar o teto; (d) comparar com baselines publicados (AVeriTeC; Delucis et al. 2025).

### 4.1 Como exatamente se dá o treinamento

**Primeiro, o que NÃO se treina:** ninguém treina um modelo do zero. Trabalha-se por **ajuste fino (fine-tuning)**: pega-se um modelo já pré-treinado em muito texto (que já "sabe português") e continua-se o treino numa tarefa específica, com poucos milhares de exemplos. É barato comparado ao pré-treinamento.

**Há três modelos possíveis no pipeline — mas só UM realmente precisa ser treinado:**

| Componente | Precisa treinar? | Observação |
|---|---|---|
| Classificador de veredito (4 classes) | **Sim** (obrigatório) | É o núcleo; ajuste no AVERITEC-BR |
| Bi-encoder de recuperação (busca densa) | Opcional | Modelos multilíngues prontos (`bge-m3`, `e5`) já funcionam; ajustar só se o domínio exigir |
| Reranker (cross-encoder) | Opcional | `bge-reranker-v2-m3` pronto; ajuste é refinamento |
| LLM (extração/explicação) | **Não** | Usa-se pronto (API ou local) com prompts e saída estruturada |

Ou seja: o esforço de treino concentra-se em **um classificador de texto** — tarefa bem estabelecida e viável.

**Formato dos dados de treino.** Cada exemplo do AVERITEC-BR vira um par entrada→rótulo:
- **Entrada**: `[CLS] alegação [SEP] trecho(s) de evidência [SEP]` (a evidência entra concatenada, com separadores).
- **Rótulo**: uma das 4 classes (suportada / refutada / inconclusiva / conflitante).

Para os componentes opcionais, o formato muda:
- **Bi-encoder** (recuperação): tripletos `(alegação, trecho positivo, trechos negativos)` com *contrastive loss* — aproxima vetores de pares corretos, afasta os errados.
- **Cross-encoder** (reranker): pares `(alegação, trecho)` com rótulo de relevância (relevante/irrelevante), *cross-entropy*.

**Passo a passo do ajuste do classificador:**
1. **Baixar e preparar**: `datasets` carrega o AVERITEC-BR; separar treino/dev/teste (e uma **partição temporal**).
2. **Tokenizar**: tokenizer do BERTimbau; truncar a entrada (ex.: 512 tokens), truncando a evidência se necessário.
3. **Definir o modelo**: `AutoModelForSequenceClassification.from_pretrained("neuralmind/bert-base-portuguese-cased", num_labels=4)`.
4. **Treinar** com `Trainer` (`transformers`) + `accelerate`:
   - Loss: *cross-entropy* (classificação multi-classe).
   - Hiperparâmetros típicos: lr `2e-5`, batch 16–32, 3–5 épocas, warmup, weight decay; **class weights** para a classe rara "conflitante".
5. **Avaliar** por época em dev (macro-F1) e usar *early stopping*.
6. **Testar** no conjunto de teste + partição temporal; gerar relatório por classe.
7. **Salvar** o modelo/pesos e versionar (Hugging Face Hub privado ou Git LFS).
8. **Iterar**: análise de erro → ajustar dados (ex.: aumentar classe conflitante com o desenho do Emergent) → repetir.

**Se faltar GPU (LoRA / feature-based):** com `peft` (LoRA) treina-se só uma fração dos parâmetros, reduzindo muito a memória; ou congela-se o encoder e treina-se apenas a **cabeça de classificação** (mais rápido, um pouco menos preciso). Alternativa clássica: extrair embeddings do BERTimbau e treinar uma regressão logística sobre eles — baseline forte e quase sem GPU.

**Custo de tempo (ordem de grandeza):** ajustar um encoder base em ~4,5 mil exemplos leva de **minutos a ~1 hora** numa GPU única; avaliar é instantâneo. Não é um treino de dias.

### 4.2 Infraestrutura necessária

**Regra prática: só o ajuste do classificador exige GPU; o resto roda em CPU.**

| Etapa | Hardware | Comentário |
|---|---|---|
| Extração de alegações (LLM) | API (nuvem) ou CPU/GPU local | API não exige hardware; local: `Ollama` + `qwen2.5:7b` pede ~8–16 GB RAM/VRAM |
| Recuperação (BM25, embeddings, FAISS) | **CPU** | Indexar e buscar em CPU é viável; GPU só acelera embeddings em massa |
| Ajuste do classificador | **GPU** (obrigatório na prática) | BERTimbau base ~8–16 GB VRAM; large ~16–24 GB; com LoRA, menos |
| Explicação (LLM) | API ou local | Idem extração |
| Interface web (Streamlit) | CPU | Roda em qualquer máquina; publicável no HF Spaces |

**Opções de GPU para a residência:**
- `Google Colab` (T4 grátis; Pro com A100/L4) — suficiente para BERTimbau base/large.
- `Kaggle Notebooks` — GPU grátis (~30 h/semana), ótimo para ajuste e experimentos.
- Cluster do **Instituto Eldorado**, se disponível — ideal para rodadas maiores e reprodutibilidade.
- Mac local (MPS): roda modelos base, porém mais lento; serve para prototipagem, não para muitos experimentos.

**Armazenamento e ambiente:** datasets somam poucos GB; use `Hugging Face datasets` e cache local. Ambiente Python 3.11 com `uv`/`conda`; `torch`, `transformers`, `datasets`, `sentence-transformers`, `faiss-cpu`, `rank_bm25`, `peft`, `scikit-learn`. Versionamento de dados com `DVC` (opcional) e código no GitHub. Tracking de experimentos com `Weights & Biases` (plano acadêmico gratuito) ou `TensorBoard`.

**Resumo prático:** com **uma GPU gratuita (Colab/Kaggle)** dá-se conta do ajuste do classificador; os demais componentes usam modelos prontos (CPU) e um LLM via API. A infraestrutura não é o gargalo do projeto — o gargalo é a qualidade/cobertura da evidência e a avaliação honesta.

## 5. Camada 4 — Explicação ancorada e interface web

- **Explicação**: LLM **estritamente grounded** — recebe o veredito do classificador + trechos de evidência e redige a justificativa **com citações**, sem poder alterar a classe. Guardar (veredito do modelo, texto gerado, evidências) para auditoria.
- **Interface web mínima**:
  - `Streamlit` (mais rápido para demo) ou `Gradio`; se precisar de API separada, `FastAPI` + front simples.
  - Dois perfis de uso: **submissão** (publicador verificado) e **leitura** (usuário comum) — a aprovação de publicadores fica fora do MVP (conjunto pré-aprovado). — [revisão 2026-09-27]
  - Exibição: classe da notícia, alegações destacadas, evidências com fonte/data, explicação — **sem veredito seco**, em layout simétrico favor/contra, exibindo também refutadas/inconclusivas com seus argumentos.
  - Publicação da demo: **Hugging Face Spaces** (grátis) ou máquina do Instituto.

## 6. Avaliação

Protocolo proposto pelo próprio relatório do grupo (§5.2), a ser implementado:
- **Macro-F1 por classe**, com relatório isolado da distinção **refutada vs. inconclusiva**.
- **Desempenho na classe de evidências conflitantes** (casos de maior interesse público).
- **Taxa de abstenção calibrada**: curva risco-cobertura + ECE (`netcal` ou implementação própria com `sklearn.calibration`).
- **Partição temporal de teste** (mitiga deriva conceitual); fontes: `Fato ou Boato` (TSE), filtros por período.
- **Checagem de contaminação**: time-aware splits; ressalva explícita sobre LIAR/FEVER.
- **Comparação qualitativa com a Fátima** (Aos Fatos).
- **Ferramentas**: `scikit-learn`, `pandas`, `matplotlib`; tracking com `Weights & Biases` (plano acadêmico) ou `TensorBoard`.

## 7. Infraestrutura e stack

- Python 3.11; ambientes com `uv` ou `conda`; `pytorch` (MPS no Mac ou CUDA).
- GPU: `Google Colab Pro`, `Kaggle` (GPU grátis), ou cluster do Instituto Eldorado.
- Dados/versionamento: `Hugging Face datasets`; `DVC` opcional; `Git` + GitHub.
- Contêiner opcional: `Docker`.

## 8. Cronograma faseado (com gates)

> Ajustar ao calendário real da residência — prazo ainda não registrado (`[NEEDS CLARIFICATION]`).

| Fase | Entrega | Gate de saída |
|---|---|---|
| F0 — Setup (dias) | Ambiente, acesso a dados, esqueleto do repo | Rodar baseline classe majoritária |
| F1 — Extração + recuperação (semanas) | Pipeline claims→evidência sobre FactCenter/AVeriTeC | Recall@k aceitável em amostra |
| F2 — Classificador (semanas) | Encoder ajustado no AVERITEC-BR | Macro-F1 vs. baselines publicados |
| F3 — Explicação + UI (semanas) | Demo Streamlit com explicação ancorada | Fluxo ponta a ponta em notícia real |
| F4 — Avaliação (semanas) | Métricas por classe, abstenção, partição temporal, comparação Fátima | Relatório de avaliação |
| F5 — Escrita (semanas) | Relatório final e apresentação | — |

**Degradação planejada**: se F2/F3 atrasarem, cortar a camada 1 generativa (usar segmentação por regras) e a explicação generativa (exibir evidências cruas) → Opção A do `concept.md`, mantendo o veredito ancorado.

## 9. Riscos técnicos e mitigação

| Risco | Mitigação |
|---|---|
| LLM alucina na explicação e contradiz o veredito | Explicação condicionada ao veredito; validação de consistência; citações obrigatórias |
| Base de evidência em inglês (AVeriTeC) | Priorizar FactCenter + Wikipédia PT; transferência cross-lingual |
| Classe "conflitante" rara | Aumento com desenho do Emergent; casos de teste curados |
| Licença CC BY-NC do AVeriTeC | Uso só acadêmico; checar licença junto aos autores do AVERITEC-BR |
| Custo/latência de LLM | LLM local (Qwen2.5 via Ollama) para extração; cache |
| Contaminação / atalhos | Partição temporal; avaliar em domínio distinto do treino |
| Deriva conceitual | Reescolha periódica de evidência; teste temporal contínuo |

## 10. Assunções a validar

- Prazo real da residência e disponibilidade de GPU.
- Acesso efetivo ao AVERITEC-BR/LIAR-BR e ao FactCenter (download + licença).
- Escolha entre LLM via API (custo) ou local (hardware).
- Orçamento de tempo do grupo para as 3 camadas (justifica o appetite `large`).