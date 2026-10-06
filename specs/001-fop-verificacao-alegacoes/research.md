# Research: FOP — decisões técnicas (Fase 0)

**Feature**: `001-fop-verificacao-alegacoes` | **Date**: 2026-09-27 | **Plan**: [plan.md](./plan.md)

Decisões realinhadas ao `fluxo-ia-fop-v3.md` (plano definitivo do Grupo 5). Formato: **Decision / Rationale / Alternatives considered**. Regra de ouro preservada: **o veredito não é decidido por LLM**.

## 1. Seleção de frases verificáveis

- **Decision**: no MVP, **regras + spaCy** (`pt_core_news_lg`): manter frases com número, data, nome próprio (NER) ou verbo de declaração; ordenar por nº de sinais; limite `max_alegacoes = 10`; registrar quantas frases ficaram de fora.
- **Rationale**: simples, auditável e sem custo de LLM; opiniões e descrições não verificáveis geram ruído. O corte é exibido ao publicador (transparência).
- **Alternatives considered**: BERTimbau ajustado para checabilidade (**incremento**, exige guia de anotação e kappa); XLM-R/mDeBERTa ajustados em outras línguas (não há tarefa de checabilidade em PT conhecida).

## 2. Descontextualização e atribuição

- **Decision**: NER + substituição de referências (só com candidato único; senão `ambigua = true`), detecção de atribuição por padrões (`fonte_citada`, `declaracao`, `nenhuma`), conversão de datas relativas e flag `sensivel_ao_tempo`.
- **Rationale**: alegação autocontida é pré-requisito da busca; a atribuição define **a quem o veredito se refere** (veículo, declaração de X, ou dado atribuído a fonte) — exigência central do v3 (W3, N1).
- **Alternatives considered**: LLM neste passo (resolveria melhor, mas viola "LLM só no fim" e a confidencialidade W12) — testar apenas se as regras falharem muito (E6/E7); BERTimbau-NER com corpus HAREM.

## 3. Base de checagens (P1)

- **Decision**: acervo de checagens de agências (Google Fact Check Tools API/padrão ClaimReview, páginas de agências, Fato ou Boato, FactCenter quando obtido), com mapeamento "veredito da agência → 4 classes", deduplicação por alegação e agrupamento quando várias agências checam o mesmo (vereditos divergentes → `conflitante`/`agencias_divergem`), embedding único e snapshot incremental diário.
- **Rationale**: reaproveitar veredito humano é o caminho mais confiável e barato; guarda-se apenas alegação curta, metadados e link.
- **Alternatives considered**: só coleta própria (mais custo/erro); sem FactCenter; pgvector/Chroma no lugar do FAISS; `paraphrase-multilingual-MiniLM-L12-v2` como ablação.

## 4. Base de evidência (P2)

- **Decision**: no MVP, **conteúdo público de fontes oficiais** (IBGE, INEP, TSE, Diário Oficial, gov.br, portais de dados abertos), com licença conferida; corte em trechos de ~150 palavras (sobreposição ~30), índices BM25 e vetorial, snapshot. Na avaliação, o *knowledge store* do AVeriTeC **com filtro anti-vazamento**.
- **Rationale**: evita risco autoral e de confidencialidade (texto de veículos só após parecer jurídico); fontes oficiais são estáveis e auditáveis. A ausência de cobertura vira "inconclusiva" — comportamento correto.
- **Alternatives considered**: web em tempo real (Brave Search/Google PSE) só para texto já publicado (W12); usar texto de agências/veículos antes do parecer (bloqueado).

## 5. Casamento com checagens existentes

- **Decision**: buscar `top_k_checagens` por embedding, exigir **NLI bidirecional** acima de `limiar_equivalencia` para casar, aplicar filtro temporal (alegações sensíveis ao tempo só herdam checagens dentro de `idade_max_checagem_meses`) e, em caso de casamento múltiplo divergente, marcar `conflitante`/`agencias_divergem`. Cache por alegação+snapshots.
- **Rationale**: herda veredito humano com segurança; o NLI bidirecional reduz casamentos falsos (erro mais grave do passo).
- **Alternatives considered**: NLI unidirecional (mais falso-positivo); similaridade apenas (insuficiente); `multilingual-MiniLMv2` (mais rápido, sem PT no treino).

## 6. Recuperação de evidência (funil)

- **Decision**: BM25 (`top_k_bm25 = 50`) + busca densa (`top_k_denso = 50`) fundidas por **RRF** (até 100 trechos); filtros (remover trechos da própria notícia; na avaliação, remover domínios de agências e documentos posteriores à alegação; em produção, penalizar datas fora da `janela_tempo_meses` quando sensível ao tempo; priorizar domínio da fonte citada); `top_n_final = 5`.
- **Rationale**: híbrida cobre números/nomes e paráfrases; orçamento de latência controlado; a base é a única origem de evidência.
- **Alternatives considered**: reranqueador `BAAI/bge-reranker-v2-m3` sobre `top_n_rerank = 30` (**incremento**; é o componente mais lento — E9); cross-encoders mMARCO.

## 7. Veredito (NLI em janelas + regras)

- **Decision**: dividir cada trecho em janelas de 1–2 frases; NLI (`MoritzLaurer/mDeBERTa-v3-base-xnli-multilingual-nli-2mil7`) com a janela como premissa e a alegação como hipótese; guardar maior `p_confirma`/`p_contradiz` por trecho; classificar trechos fortes por `limiar_suporta`/`limiar_refuta`; agregar por regras (suportada / refutada / conflitante / inconclusiva) — **a classe inconclusiva não recebe confiança** (evita circularidade).
- **Rationale**: componentes auditáveis, sem LLM; alinhado à evidência de que o ganho vem da evidência recuperada; tokens com premissa primeiro e `truncation="only_first"`.
- **Alternatives considered**: classificador ajustado sobre (alegação, 5 trechos) no formato AVERITEC-BR (**alternativa**; se adotado, limiares migram para o dev); verificador numérico próprio (**incremento** W8: só decide com entidade+unidade+período coincidentes; discordância → inconclusiva + revisão).

## 8. Calibração, abstenção e revisão

- **Decision**: calibração (Platt por classe ou classe como variável; temperature scaling) ajustada no **train-B**; `limiar_abstencao` escolhido no **train-A** pela curva risco-cobertura; veredito abaixo do limiar vira inconclusiva ("baixa confiança"). Fila de revisão para: toda refutada; conflitante com pessoa; `ambigua`; conflito numérico×NLI; mudança de veredito entre versões (C4). Log de auditoria com snapshots/versões.
- **Rationale**: torna a abstenção uma decisão medida; mantém humano nas decisões danosas; evita punir notícia sem revisão.
- **Alternatives considered**: limiar direto sem calibrador (menos confiável); isotônica (sobreajusta com poucos dados); revisão amostral de suportadas (**incremento** C5).

## 9. Explicação

- **Decision**: **templates** no MVP, por origem e tipo de atribuição (agência: "Checada por [agência] em [data]…"; fluxo próprio: "Segundo [fonte] ([data]): '[janela decisiva]'…"; declaração: "Declaração de [X]:"; fonte citada: "A notícia atribui este dado a [fonte]:"; inconclusiva sem trechos: mensagem de ausência de evidência). Trecho exibido limitado a `max_palavras_trecho_exibido = 25`.
- **Rationale**: legível, previsível e sem risco de alucinação; atende FR-011 e o layout simétrico.
- **Alternatives considered**: LLM local (7–8B via Ollama/llama.cpp) como **incremento**, com frases factuais filtradas por NLI e conclusão sempre do template; API externa só com retenção zero — inviável no MVP por confidencialidade (W12).

## 10. Embedding único e índice

- **Decision**: um único modelo de embedding para P1, P2 e casamento: `intfloat/multilingual-e5-base` **ou** `BAAI/bge-m3` (escolha na EDA); FAISS para o índice vetorial; vetores normalizados; prefixos `query:`/`passage:` se for o e5; trocar de modelo exige reindexar.
- **Rationale**: consistência entre bases e consulta; custo computacional gerenciável em CPU.
- **Alternatives considered**: modelos por base (complexidade); OpenSearch (escala; Docker) no lugar do `rank_bm25`.

## 11. Persistência, snapshots e retenção

- **Decision**: **SQLite** para metadados (checagens, trechos, alegações, vereditos, fila de revisão, log de auditoria) + FAISS em arquivo; snapshots `snapshot_p1`/`snapshot_p2` e `versao_config` registrados por veredito. O **texto da notícia é transitório**: mantido apenas durante a verificação e a revisão pendente, descartado após a publicação/conclusão (FR-025).
- **Rationale**: cumpre a decisão de retenção revista, viabiliza versões/diff para o revisor (C4) e mantém auditabilidade (FR-015).
- **Alternatives considered**: Postgres (overkill no MVP); retenção permanente do texto (contraria a decisão de privacidade); apenas arquivos JSON (pior consulta/concorrência).

## 12. Ferramentas e ambiente

- **Decision**: Python 3.11; `spaCy` (pt_core_news_lg), `dateparser`, `trafilatura`, `datasketch`, `rank_bm25`, `sentence-transformers` + FAISS, `transformers` (NLI mDeBERTa), `scikit-learn` (calibradores), `FastAPI` + Uvicorn, `Streamlit`, `pandas`. **Execução sem GPU**; GPU só para ajuste/calibração quando disponível; `Ollama` opcional para o incremento de explicação.
- **Rationale**: todo o fluxo de execução é CPU-friendly (NLI base + FAISS), compatível com a demo em Hugging Face Spaces; bibliotecas abertas mantêm custo ~zero.
- **Alternatives considered**: LLM obrigatório (contraria confidencialidade/custo); OpenSearch em produção com maior volume.

## 13. Avaliação (protocolo anti-vazamento)

- **Decision**: splits do AVERITEC-BR (train dividido em train-A para limiares/abstenção e train-B para calibração; dev para variantes; teste **uma vez** no fim); na avaliação, remover domínios de agências e documentos posteriores à alegação; **deduplicar o teste contra P1** e avaliar o passo de casamento à parte (E2); métricas com e sem o passo 3, com bootstrap. Métricas principais: F1 por classe, precisão de refutada, confusão refutada × inconclusiva, erro de calibração por classe e taxa de erro nos casos decididos vs. abstenção.
- **Rationale**: evita vazamento (respostas prontas na base) e reporta a **validade externa** (a precisão de refutada medida no teste não estima produção — reajuste no piloto com E11).
- **Alternatives considered**: avaliar sem deduplicar (inflaria resultados); medir só acurácia global.

## 14. Licenças e conformidade

- **Decision**: uso acadêmico não comercial; conferir licenças de cada fonte; **nenhuma consulta com texto não publicado sai para serviços externos** (W12); **parecer jurídico** (direito autoral, reuso de vereditos de agências e LGPD) é **portão do piloto**; texto de veículos/agências só entra em P2 depois do parecer.
- **Rationale**: o piloto é uso real; evita risco jurídico e reputacional (refutada referente a pessoas).
- **Alternatives considered**: usar dados de terceiros sem licença (risco); reter texto (contraria retenção).

## 15. Riscos técnicos e mitigação

| Risco | Mitigação |
|---|---|
| Casamento falso herda veredito errado | NLI bidirecional + filtro temporal + conjunto de contraste (E2) |
| Sem trecho relevante → inconclusiva demais (notícias regionais) | Cobertura oficial limitada é limitação declarada; revisar exibição de "inconclusiva" (decisão do grupo) |
| NLI inflado por máximo entre janelas | Limiares ajustados com o mesmo janelamento; reajustar se o janelamento mudar |
| Vazamento na avaliação | Remover agências e documentos posteriores; dedup do teste contra P1 |
| Mudança de veredito por reformulação | Toda mudança refutada → suportada/inconclusiva vai para revisão (C4) |
| Injeção de instruções nos trechos (só afeta incremento LLM) | Trechos entre delimitadores; LLM não decide veredito; NLI filtra frases |
| Latência do reranqueador | Reranker como incremento; benchmark E9 antes de fixar |
| Deriva conceitual | Snapshots + reavaliação periódica; reajuste no piloto (E11) |

## 16. Todo NEEDS CLARIFICATION do Technical Context resolvido?

Sim — linguagem, dependências, storage, testes, plataforma, tipo de projeto, desempenho, constraints e escala definidos acima, alinhados ao `fluxo-ia-fop-v3.md`. Permanecem **pendências de EDA** (não bloqueiam o plano): tamanhos/licença dos splits do AVERITEC-BR e existência de data por documento no knowledge store de avaliação.
