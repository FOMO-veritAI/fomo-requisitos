# FOP — Requisitos Funcionais e Não Funcionais (documento consolidado)

**Gerado em**: 2026-09-27
**Projeto**: TAKTA / FOP — Residência em IA e Análise de Dados (Instituto Eldorado + UnB/FGA) · Grupo 5
**Natureza**: documento de **fusão**, para leitura e conferência. As fontes de verdade continuam sendo os arquivos originais:
`specs/001-fop-verificacao-alegacoes/spec.md` (FR), `.../plan.md` (NFR) e `fluxo-ia-fop-v3.md` (desenho).

## Fontes

| Abrev. | Documento | O que fornece |
|---|---|---|
| [SPEC] | `specs/001-fop-verificacao-alegacoes/spec.md` | Requisitos funcionais, user stories, critérios de sucesso, clarificações |
| [PLAN] | `specs/001-fop-verificacao-alegacoes/plan.md` | Requisitos não funcionais, contexto técnico, metas |
| [V3] | `fluxo-ia-fop-v3.md` | Desenho definitivo: preparação P1–P3, execução 1–7, parâmetros, avaliação |
| [DEC] | `.specify/assessments/takta/decision.md` | Veredito e handoff do assessment |

---

# Parte I — Requisitos Funcionais (FR)

## 1. Papéis, submissão e versões

### FR-001 — Submissão restrita a publicador verificado
- **Requisito**: O sistema DEVE permitir que apenas publicadores verificados submetam notícias para verificação.
- **Origem [SPEC, US1]**: "Um publicador verificado (veículo de comunicação ou organização editorial aprovado pela administração da plataforma) submete uma notícia e recebe, antes de publicá-la..."
- **Origem [V3, passo 1]**: "Pré-condições: veículo verificado; texto em português (detecção de idioma); texto não vazio."
- **Verificação**: SC-007; edge case "Usuário comum (não verificado) tenta submeter: a submissão é negada".

### FR-016 — Negar submissão de não verificados
- **Requisito**: O sistema DEVE negar a submissão de notícias por usuários não verificados.
- **Origem [SPEC, clarificação]**: "Quem pode submeter notícias para verificação na FOP? → Apenas publicadores verificados...; usuários comuns apenas visualizam e interagem."
- **Origem [V3, passo 1]**: "Pré-condições: veículo verificado..."
- **Verificação**: `403 publisher_not_verified` no contrato HTTP; SC-007.

### FR-017 — Usuários comuns visualizam
- **Requisito**: O sistema DEVE permitir que usuários comuns visualizem as notícias publicadas, suas classificações e suas evidências.
- **Origem [SPEC, US2]**: "Usuários comuns visualizam as notícias publicadas pelos publicadores verificados..."
- **Origem [V3, finalidade]**: "para que o veículo possa corrigir a notícia e que o leitor avalie a informação por conta própria, sem que a IA decida por ele."
- **Verificação**: edge case "no modo leitura, não existe ação de submissão".

### FR-018 — Distinção de papéis
- **Requisito**: O sistema DEVE distinguir os papéis de publicador verificado, usuário comum, revisor e administrador da plataforma.
- **Origem [SPEC, US4]**: "Um revisor designado pela plataforma recebe os casos sensíveis... Quem revisa (moderação, veículo ou ambos) é decisão da administração."
- **Origem [V3, passo 6.3]**: "Encaminhar para revisão humana: toda refutada; toda conflitante envolvendo pessoa...".
- **Observação**: o papel `admin` (aprovar/revogar publicadores) está **fora do MVP**.

### FR-022 — Correção e ressubmissão (até 3 versões)
- **Requisito**: O sistema DEVE permitir ao publicador corrigir e ressubmeter a mesma notícia, até 3 versões, mantendo o histórico ligado.
- **Origem [SPEC, clarificação]**: "Quantas ressubmissões são permitidas? → Até 3 por notícia no MVP; as versões ficam ligadas no histórico..."
- **Origem [V3, passo 6.3/C4]**: "toda alegação cujo veredito mude de refutada para suportada ou inconclusiva entre versões da mesma notícia... O revisor recebe a diferença entre as versões do texto."
- **Verificação**: endpoint `POST /verifications/{id}/versions`; SC-010.

### FR-023 — Mudança de veredito entre versões vai à revisão
- **Requisito**: O sistema DEVE detectar mudança de veredito entre versões da mesma notícia e encaminhá-la à revisão quando a mudança for de refutada para suportada ou inconclusiva.
- **Origem [V3, passo 6.3/C4]**: "uma correção legítima do veículo é o comportamento desejado, e o que se quer barrar é a reformulação que muda só a forma."
- **Verificação**: SC-010; E10 (red team: reformulação até obter "suportada").

### FR-021 — Bloqueio de publicação com pendência de revisão
- **Requisito**: O sistema DEVE impedir a publicação enquanto houver caso pendente de revisão, notificando o publicador.
- **Origem [SPEC, clarificação]**: "A notícia pode ser publicada com revisão pendente? → Não no MVP; enquanto houver caso pendente, a notícia não é publicada e o publicador é notificado."
- **Origem [V3, seção 8, item 2]**: decisão em aberto do grupo — no MVP adotou-se o padrão conservador.
- **Verificação**: `publication_allowed = false`; SC-008.

## 2. Pipeline de verificação

### FR-002 — Seleção de frases verificáveis
- **Requisito**: O sistema DEVE selecionar as frases verificáveis da notícia (por regras: número, data, nome próprio ou verbo de declaração), limitadas a 10 alegações principais, registrando quantas ficaram de fora.
- **Origem [V3, passo 1]**: "manter frases com pelo menos um destes sinais: número, data, nome próprio (NER) ou verbo de declaração... manter até `max_alegacoes` (padrão 10). Registrar quantas frases ficaram de fora do corte, para exibir ao veículo (W16)."
- **Incremento [V3]**: BERTimbau ajustado para checabilidade (exige guia de anotação e kappa).

### FR-003 — Descontextualização e atribuição
- **Requisito**: O sistema DEVE descontextualizar cada alegação e classificar sua atribuição (`fonte_citada`, `declaracao` ou `nenhuma`), resolvendo datas relativas a partir da data da notícia.
- **Origem [V3, glossário]**: "`fonte_citada`: a notícia usa uma fonte como autoridade para um dado... O veredito é **do veículo**... `declaracao`: a notícia relata a fala de alguém... Verifica-se Y, e o resultado é exibido como 'declaração de X'. `nenhuma`: o veículo afirma diretamente."
- **Origem [V3, passo 2]**: "o veículo não pode ser responsabilizado pela fala que apenas relata, e continua responsável pelo dado que atribui a uma fonte (W3, N1)."

### FR-004 — Sensibilidade ao tempo
- **Requisito**: O sistema DEVE marcar se a alegação é sensível ao tempo (número, cargo ou índice sem data explícita).
- **Origem [V3, passo 2.5/C7]**: "`sensivel_ao_tempo = true` se a alegação tiver número, cargo ou índice **sem data explícita**, ou contiver 'atual', 'hoje', 'agora'... Alegações datadas ('em 2018, o PIB cresceu 1,8%') não são sensíveis ao tempo."

### FR-005 — Casamento com checagens existentes
- **Requisito**: O sistema DEVE casar a alegação com checagens existentes (similaridade + verificação bidirecional de equivalência + filtro temporal) e herdar o veredito humano quando houver casamento.
- **Origem [V3, passo 3]**: "Verifica se a alegação já foi checada por uma agência e, se foi, reaproveita o veredito... NLI nos dois sentidos: só há casamento se a alegação implicar a checagem e a checagem implicar a alegação... Filtro temporal... Se várias checagens casarem com vereditos diferentes, o resultado é 'conflitante' com motivo `agencias_divergem`."
- **Verificação**: E2 (conjunto de contraste) — taxa de casamentos falsos.

### FR-006 — Recuperação de evidência
- **Requisito**: O sistema DEVE recuperar evidências externas por busca lexical e semântica com fusão, **nunca** usando a própria notícia como evidência.
- **Origem [V3, passo 4]**: "Os trechos vêm da base P2, **nunca da notícia**... BM25 (`top_k_bm25` = 50) + busca por significado (`top_k_denso` = 50) → fusão por *Reciprocal Rank Fusion*... `top_n_final` = 5."
- **Incremento [V3]**: reranqueador `BAAI/bge-reranker-v2-m3` (componente mais lento; medir latência, E9).

### FR-007 — Veredito em quatro classes
- **Requisito**: O sistema DEVE classificar cada alegação em exatamente uma das quatro categorias: suportada, refutada, inconclusiva ou conflitante.
- **Origem [V3, passo 5]**: "NLI por janela: a janela é a premissa, a alegação é a hipótese... Agregar por regras: [suportada / refutada / conflitante / inconclusiva]."
- **Regra de ouro [V3, passo 5]**: "é aqui que a FOP decide, com componentes auditáveis e **sem LLM**."

### FR-009 — Evidências com fonte, data e trecho decisivo
- **Requisito**: O sistema DEVE apresentar, para cada classificação, as evidências com fonte, data e o trecho decisivo.
- **Origem [V3, passo 7]**: "o texto exibido é a **janela decisiva**, cortada em `max_palavras_trecho_exibido` (padrão 25)."
- **Verificação**: SC-002 ("para 100% das classificações exibidas, ao menos uma evidência com fonte e data").

### FR-011 — Explicação neutra e com atribuição
- **Requisito**: O sistema DEVE apresentar uma explicação em linguagem simples, ancorada nas evidências e neutra, identificando a quem o resultado se refere (veículo, declaração de X, ou dado atribuído à fonte).
- **Origem [V3, passo 7]**: "`tipo_atribuicao = declaracao`: o texto começa com 'Declaração de [X]:'. `tipo_atribuicao = fonte_citada`: 'A notícia atribui este dado a [fonte]:'. O resultado continua sendo da notícia."
- **Origem [V3, passo 7, inconclusiva]**: "Não encontramos evidência suficiente para verificar esta alegação."

### FR-012 — Corte temporal
- **Requisito**: O sistema DEVE considerar apenas evidências disponíveis até o momento da submissão, incluindo idade máxima para herança de checagens antigas em alegações sensíveis ao tempo.
- **Origem [SPEC, edge case]**: "Evidência publicada após o momento da submissão: não pode ser usada."
- **Origem [V3, passos 3 e 6]**: "`idade_max_checagem_meses` (padrão 12)"; na avaliação: "remover documentos publicados depois da data da alegação".

### FR-013 — Declarar incerteza e conflito
- **Requisito**: O sistema DEVE declarar explicitamente incerteza ou conflito, com o motivo registrado.
- **Origem [V3, passo 5]**: "Nenhum forte → Inconclusiva... Para a inconclusiva, registra-se no log apenas a maior probabilidade decisiva observada... Ela não é calibrada nem exibida."
- **Origem [V3, passo 6]**: "Abaixo de `limiar_abstencao`, o veredito vira inconclusiva com motivo 'baixa confiança'."

### FR-014 — Operar em português do Brasil
- **Requisito**: O sistema DEVE operar em português do Brasil.
- **Origem [SPEC, edge case]**: "Conteúdo em idioma diferente do português do Brasil: fora do escopo desta versão; o sistema informa."
- **Origem [V3, passo 1]**: detecção de idioma como pré-condição.

### FR-025 — Retenção transitória do texto
- **Requisito**: O sistema DEVE manter o texto da notícia apenas durante a verificação e a revisão pendente, descartando-o após a publicação ou a conclusão sem pendências.
- **Origem [SPEC, clarificação revisada]**: "Não persistir de forma permanente; manter apenas durante a verificação e eventual revisão pendente, descartando após a publicação/conclusão (revisado em 2026-09-27 para viabilizar versões e revisão)."
- **Origem [V3, P1]**: "Guarda-se só a alegação curta, os metadados e o link, nunca o artigo inteiro."
- **Nota**: revisa a decisão anterior de "não persistir o texto", porque o diff entre versões (C4) exige retê-lo durante a revisão.

## 3. Incerteza, integridade e exibição

### FR-008 — Ausência de evidência nunca é falsidade
- **Requisito**: O sistema NÃO DEVE tratar ausência de evidência como falsidade.
- **Origem [SPEC, US1]**: "a classificação é 'inconclusiva', nunca 'refutada' por ausência de prova."
- **Origem [V3, seção 5]**: P2 "Se a base não contiver o fato, o melhor resultado possível é 'inconclusiva'."

### FR-010 — Refutadas permanecem visíveis
- **Requisito**: O sistema DEVE manter notícias refutadas visíveis após a publicação, sem bloqueá-las ou ocultá-las.
- **Origem [SPEC, US2]**: "notícias classificadas como refutadas permanecem visíveis, acompanhadas da classificação e dos argumentos/evidências que a sustentam."
- **Origem [esboco.txt]**: "Notícias identificadas como falsas ou fortemente contraditas pelas evidências não serão distribuídas normalmente..." — **revisado pelo grupo**: exibir com classificação (estímulo ao pensamento crítico).

### FR-019 — Apresentação simétrica
- **Requisito**: O sistema DEVE apresentar, por alegação, as evidências favoráveis e contrárias de forma simétrica (lado a lado), com explicação neutra.
- **Origem [SPEC, clarificação]**: "Lado a lado por alegação — evidências favoráveis e contrárias em colunas simétricas, com explicação neutra."
- **Origem [V3, passo 7]**: exibição centrada na janela decisiva e nas fontes.

### FR-015 — Registro de auditoria
- **Requisito**: O sistema DEVE registrar, para auditoria, snapshots das bases, versão de parâmetros e modelos, substituições feitas e resultado de cada verificação.
- **Origem [V3, passo 6.4]**: "Registrar no log de auditoria: `noticia_id`, `versao_texto`, `snapshot_p1`, `snapshot_p2`, versões dos modelos, `versao_config`, substituições feitas, tipo de atribuição e o resultado."

### FR-020 — Fila de revisão humana
- **Requisito**: O sistema DEVE encaminhar à fila de revisão os casos sensíveis: toda refutada; conflitante envolvendo pessoa; alegação ambígua; conflito entre verificador numérico e o veredito; e mudança de veredito entre versões.
- **Origem [V3, passo 6.3]**: "toda **refutada**; toda **conflitante** envolvendo pessoa...; toda alegação com `ambigua = true`...; todo conflito entre o verificador numérico e o NLI; toda alegação cujo veredito mude..."
- **Origem [V3, passo 6]**: "mantém o humano nas decisões de maior dano."

### FR-024 — Confidencialidade do texto não publicado
- **Requisito**: O sistema NÃO DEVE enviar o texto não publicado a serviços externos no MVP.
- **Origem [V3, passo 4.3/W12]**: "nenhuma consulta com texto não publicado sai para serviços externos no MVP."
- **Origem [V3, passo 7.3]**: "LLM por API externa só com retenção zero contratual, porque o texto ainda não foi publicado (W12)."
- **Consequência no MVP**: explicação por **templates**; LLM local apenas como incremento.

---

# Parte II — Requisitos Não Funcionais (NFR)

> Consolidados no [PLAN]; metas genéricas de MVP acadêmico, revisáveis no piloto.

## Desempenho

### NFR-001 — Latência da verificação
- **Requisito**: verificação assíncrona de notícia típica (≤1.000 palavras, ≤10 alegações) com p95 ≤120 s e p50 ≤60 s em CPU; publicador notificado ao concluir.
- **Origem [PLAN, Performance Goals]**: "notícia típica (≤1.000 palavras, ≤10 alegações) em ≤2 min (p95, provisório)".
- **Origem [SPEC, SC-005]**: "o resultado é emitido em até 2 minutos (p95), provisoriamente."
- **Origem [V3, seção 8, item 9]**: meta de latência ainda **em aberto** no v3 — este valor é provisório até o benchmark (E9).

### NFR-002 — Orçamento de tempo por etapa
- **Requisito**: seleção de frases ≤5 s; casamento ≤5 s; recuperação ≤20 s; veredito/NLI ≤60 s; calibração + explicação ≤15 s.
- **Origem [V3, seção 5 Operação]**: "Benchmark obrigatório de tempo e custo por notícia com 10 alegações, em CPU e GPU, antes de fixar `top_n_rerank` e o número de janelas (E9)."
- **Origem [V3, passo 4.3]**: "O reranqueador... é o componente mais lento do fluxo: medir latência (E9)."

## Escalabilidade e disponibilidade

### NFR-003 — Concorrência e volume
- **Requisito**: 10 submissões simultâneas e ~50/dia sem degradar o p95 em >20%; fila assíncrona absorve pico de 10×.
- **Origem [PLAN, Scale/Scope]**: "dezenas de notícias/dia".
- **Origem [V3, Operação]**: "a notícia entra numa fila e o veículo é avisado quando o resultado fica pronto" + E9 "comportamento num pico de 10 vezes o volume médio".

### NFR-004 — Disponibilidade da demo
- **Requisito**: ≥95% na janela de avaliação; reinício automático; estado das verificações persistido localmente.
- **Origem [PLAN, Target Platform]**: "demo publicada em Hugging Face Spaces (CPU)".
- **Origem [V3, Operação]**: fila assíncrona + cache invalidado por snapshot.

## Confiabilidade e observabilidade

### NFR-005 — Falha de componente não é fatal
- **Requisito**: falha de recuperação/NLI/explicação → inconclusiva ou template; nenhum registro perdido; sem erro 5xx por dependência opcional.
- **Origem [V3, P2]**: "Se a base não contiver o fato, o melhor resultado possível é 'inconclusiva'."
- **Origem [V3, passo 7]**: "Se não sobrar frase válida, volta o template do MVP."

### NFR-006 — Observabilidade
- **Requisito**: log estruturado por verificação (JSON): tempo por etapa, classe, abstenção, origem e versões; métricas p50/p95, abstenção e fila de revisão; retenção de logs 90 dias.
- **Origem [V3, passo 6.4]**: log de auditoria por verificação.
- **Origem [V3, passo 6.3]**: "Monitorar a taxa de abstenção e o tamanho da fila de revisão."

### NFR-007 — Auditabilidade e reprodutibilidade
- **Requisito**: veredito ligado a `snapshot_p1`/`snapshot_p2`/`versao_config`; mesmo snapshot + seed → mesmo resultado; registro imutável.
- **Origem [V3, P3]**: "Seeds fixas"; "Recalibrar quando: um modelo for trocado..."
- **Origem [V3, glossário]**: "**Snapshot:** identificador da versão de uma base... registrado em cada veredito."

## Segurança e privacidade

### NFR-008 — Confidencialidade do texto não publicado
- **Requisito**: nenhuma chamada externa com o texto; segredos em variável de ambiente; acesso por papel; HTTPS.
- **Origem [V3, passo 4.3/W12]**: "nenhuma consulta com texto não publicado sai para serviços externos no MVP."
- **Origem [V3, P1.4.3]**: "Chave em variável de ambiente, nunca no código."

### NFR-009 — Privacidade / LGPD
- **Requisito**: texto transitório descartado após publicação/conclusão; logs sem conteúdo integral; exclusão sob solicitação; parecer jurídico antes do piloto.
- **Origem [V3, seção 8, item 7]**: "**(Pré-requisito do piloto)** Base legal (LGPD) para processar alegações sobre pessoas identificadas... Pede parecer jurídico."
- **Origem [V3, seção 8, item 11]**: risco reputacional/jurídico de exibir "declaração de [pessoa]: refutada".

## Qualidade de uso e operação

### NFR-010 — Acessibilidade
- **Requisito**: telas principais (submissão, revisão e leitura) em WCAG 2.1 nível AA: contraste ≥4,5:1, navegação por teclado, rótulos/ARIA; conteúdo em pt-BR.
- **Origem [SPEC, Assumptions]**: público "leitores de notícias em geral... com acesso via navegador" (meta de inclusão adotada pelo grupo).

### NFR-011 — Usabilidade e transparência
- **Requisito**: classificação e incerteza com fonte/data/janela; layout simétrico; atribuição explícita; leitura típica ≤3 min; nada bloqueado ao leitor.
- **Origem [V3, seção 8, item 5]**: "**(Urgente)** Como exibir 'inconclusiva' para que não seja lida como 'suspeita'? Pede teste com usuários."
- **Origem [SPEC, SC-006]**: "ao menos 70% dos participantes concordam que a apresentação não entrega conclusão pronta...".

### NFR-012 — Manutenibilidade
- **Requisito**: cobertura ≥70% nos módulos críticos (veredito, filtros temporais, autorização, revisão); suíte ≤5 min em CPU; dependências fixadas; lint.
- **Origem [PLAN, Testing]**: "pytest... avaliação offline com scikit-learn".

### NFR-013 — Portabilidade
- **Requisito**: Linux/macOS/Windows + contêiner; execução sem GPU; demo em CPU.
- **Origem [PLAN, Target Platform]**: "demo publicada em Hugging Face Spaces (CPU); ajuste/calibração em GPU (Colab/Kaggle)".
- **Origem [V3, P3]**: "No MVP, nada em P3 treina modelos" (calibração é leve).

### NFR-014 — Custo operacional
- **Requisito**: demo a custo ~zero (CPU + bibliotecas abertas); nenhuma API paga obrigatória; cache de alegações repetidas.
- **Origem [V3, Operação]**: "**Cache** de alegações repetidas (passo 3), invalidado quando mudam os snapshots."
- **Origem [V3, passo 7]**: explicação por templates no MVP (sem custo de LLM).

---

# Parte III — Rastreabilidade

## FR → User Story → Critério de sucesso → Passo do v3

| FR | US | SC | Passo v3 |
|----|----|----|----------|
| FR-001 | US1 | SC-007 | 1 (pré-condição) |
| FR-002 | US1 | SC-001 | 1 |
| FR-003 | US1/US3 | SC-006 | 2 |
| FR-004 | US1 | — | 2 (C7) |
| FR-005 | US1 | SC-003 | 3 |
| FR-006 | US1 | SC-002 | 4 |
| FR-007 | US1 | SC-001/SC-003 | 5 |
| FR-008 | US1 | SC-001 | 5 |
| FR-009 | US3 | SC-002 | 7 |
| FR-010 | US2 | SC-004 | 7 (entrega) |
| FR-011 | US3 | SC-006 | 7 |
| FR-012 | US1 | — | 3/4/6 |
| FR-013 | US1 | — | 5/6 |
| FR-014 | — | — | 1 |
| FR-015 | US4 | — | 6.4 |
| FR-016 | US1 | SC-007 | 1 |
| FR-017 | US2 | — | entrega |
| FR-018 | US2/US4 | — | 6 |
| FR-019 | US2 | SC-006 | 7 |
| FR-020 | US4 | SC-008 | 6.3 |
| FR-021 | US1/US4 | SC-008 | 6.3 |
| FR-022 | US1 | SC-010 | 6.3 (C4) |
| FR-023 | US1/US4 | SC-010 | 6.3 (C4) |
| FR-024 | US1 | SC-009 | 4.3/7.3 (W12) |
| FR-025 | US1 | — | P1/6.4 |

## NFR → Onde se verifica

| NFR | Verificação |
|-----|-------------|
| NFR-001/002 | Benchmark de carga E9; SC-005 |
| NFR-003 | E9 (pico de 10×) |
| NFR-004 | Operação da demo (HF Spaces) |
| NFR-005 | Testes de integração (falhas simuladas) |
| NFR-006 | Logs/métricas; monitorar abstenção e fila |
| NFR-007 | Log de auditoria + seeds/snapshots |
| NFR-008/009 | Revisão de chamadas externas; parecer jurídico (portão do piloto) |
| NFR-010 | Auditoria de acessibilidade (WCAG AA) |
| NFR-011 | Teste com usuários (E8/Section 8.5) |
| NFR-012 | `pytest --cov` + CI |
| NFR-013 | Execução em CPU/contêiner |
| NFR-014 | Custo da demo; cache |

## Critérios de sucesso [SPEC]

| SC | Texto |
|----|-------|
| SC-001 | 100% das notícias com alegações verificáveis recebem uma das quatro classificações, nunca falsidade por falta de evidência. |
| SC-002 | 100% das classificações exibidas têm ao menos uma evidência com fonte e data. |
| SC-003 | Refutada × inconclusiva reportadas separadamente, desempenho ≥ referências publicadas. |
| SC-004 | 100% das refutadas permanecem acessíveis após publicação, sem ocultação. |
| SC-005 | Notícia típica (≤1.000 palavras, ≤10 alegações) em ≤2 min (p95), provisório. |
| SC-006 | Enquete pós-uso: ≥70% concordam que não houve conclusão pronta e formaram sua interpretação. |
| SC-007 | 100% das submissões de não verificados negadas. |
| SC-008 | 100% dos casos sensíveis entram na fila de revisão antes de publicar. |
| SC-009 | 0% do texto não publicado enviado a serviços externos no MVP. |
| SC-010 | 100% das ressubmissões com histórico ligado e mudanças de veredito sinalizadas. |

---

# Apêndice — Parâmetros iniciais [V3, seção 4]

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
| `limiar_refuta` | 5 | (P3, train-A, precisão-alvo; reajuste no piloto E11) |
| `tolerancia_numerica` | 5 (incremento) | (P3) |
| `limiar_abstencao` | 6 | (P3, train-A; reajuste E11) |
| `taxa_revisao_suportadas` | 6 (incremento) | (grupo) |
| `max_palavras_trecho_exibido` | 7 | 25 |

## Decisões ainda em aberto [V3, seção 8]

Quem revisa (moderação/veículo/ambos) · publicação com revisão pendente · canal de contestação · nº de ressubmissões exibidas · exibição de "inconclusiva" · precisão-alvo de refutada · parecer jurídico/LGPD · índice próprio × busca web · meta de latência · política de reverificação · fidelidade da citação · taxa de revisão de suportadas.

---

**Fim do documento consolidado.** Para alterações, edite as fontes (`spec.md`/`plan.md`/`fluxo-ia-fop-v3.md`) e este consolidado deve ser regenerado.
