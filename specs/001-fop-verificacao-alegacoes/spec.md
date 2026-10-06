# Feature Specification: FOP — Verificação de alegações por IA

**Feature Branch**: `001-fop-verificacao-alegacoes`

**Created**: 2026-09-27

**Status**: Draft

**Input**: User description: "MVP da FOP (Fact On Publication), conforme `fluxo-ia-fop-v3.md`: para cada notícia submetida por um publicador verificado, entregar antes da publicação um resultado por alegação — suportada, refutada, inconclusiva ou conflitante — com evidências (fonte/data) e explicação conferida, para que o veículo corrija e o leitor avalie por conta própria. Handoff: `.specify/assessments/takta/decision.md`."

## Clarifications

### Session 2026-09-27

- Q: O que deve acontecer com o texto integral da notícia depois que a verificação termina? → A: Não persistir de forma permanente; manter apenas durante a verificação e eventual revisão pendente, descartando após a publicação/conclusão (revisado em 2026-09-27 para viabilizar versões e revisão).
- Q: Quando surgirem novas evidências depois da verificação, o sistema deve rever a classificação da notícia? → A: Não rever automaticamente; nova verificação apenas por ressubmissão do publicador (nova versão da mesma notícia).
- Q: Como a interface deve apresentar a classificação e as evidências para não induzir uma conclusão pronta? → A: Lado a lado por alegação — evidências favoráveis e contrárias em colunas simétricas, com explicação neutra.
- Q: Como medir o "estímulo ao pensamento crítico" para tornar o SC-006 verificável? → A: Enquete pós-uso (escala 1–5); meta de ao menos 70% de concordância.
- Q: Qual tamanho máximo deve ter uma "notícia típica" para o critério de resultado em até 2 minutos? → A: Até 1.000 palavras.
- Q: Quem pode submeter notícias para verificação na FOP? → A: Apenas publicadores verificados (veículos/organizações aprovados pela administração da plataforma); usuários comuns apenas visualizam e interagem.
- Q: A FOP deve ter revisão humana? → A: Sim — fila de revisão para casos sensíveis: toda refutada; conflitante envolvendo pessoa; alegação ambígua; conflito entre verificador numérico e NLI; e mudança de veredito entre versões da mesma notícia. Não há canal formal de contestação no MVP.
- Q: A notícia pode ser publicada com revisão pendente? → A: Não no MVP; enquanto houver caso pendente, a notícia não é publicada e o publicador é notificado.
- Q: Qual a meta de latência? → A: Até 2 minutos (p95) para notícia típica (≤1.000 palavras e ≤10 alegações), provisória até o benchmark de carga.
- Q: Quantas ressubmissões são permitidas? → A: Até 3 por notícia no MVP; as versões ficam ligadas no histórico; mudança de refutada para suportada/inconclusiva exige revisão.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Publicador verificado submete e recebe a verificação antes de publicar (Priority: P1)

Um publicador verificado submete uma notícia e recebe, por alegação, uma classificação em quatro categorias (suportada, refutada, inconclusiva, conflitante), com as evidências usadas (fonte, data e janela decisiva) e uma explicação neutra. Casos sensíveis ficam pendentes de revisão e a notícia não é publicada enquanto isso. O publicador pode corrigir e ressubmeter (até 3 versões).

**Why this priority**: é o núcleo do produto; sem esse fluxo a FOP não existe.

**Independent Test**: submeter uma notícia como publicador verificado e verificar que o resultado traz classes por alegação, evidências com fonte/data, explicação e status; e que notícia com caso sensível não publica até revisão.

**Acceptance Scenarios**:

1. **Given** um publicador verificado, **When** submete uma notícia, **Then** recebe, por alegação, uma das quatro classificações com evidências (fonte/data) e explicação.
2. **Given** alegações sem evidência suficiente, **When** verificadas, **Then** a classificação é "inconclusiva", nunca "refutada" por ausência de prova.
3. **Given** evidências que apontam em direções opostas, **When** verificadas, **Then** a classificação é "conflitante", com o motivo registrado.
4. **Given** uma alegação com veredito sensível (ex.: refutada), **When** a verificação termina, **Then** o caso vai para revisão e a notícia fica impedida de publicar até a decisão.
5. **Given** um publicador que corrigiu a notícia, **When** ressubmete (até 3 vezes), **Then** uma nova versão é verificada e ligada ao histórico; mudança de refutada para suportada/inconclusiva exige revisão.

---

### User Story 2 - Leitor vê notícias publicadas com classificação e evidências, sem bloqueio (Priority: P2)

Usuários comuns visualizam as notícias publicadas (após a verificação e a revisão, quando houver) com a classificação por alegação; notícias refutadas permanecem visíveis com seus argumentos e evidências, em layout simétrico (favorável × contrária). O leitor não submete notícias.

**Why this priority**: materializa o princípio pedagógico e a política de conteúdo; sem isso o produto contradiz sua premissa.

**Independent Test**: acessar, como usuário comum, uma notícia publicada refutada e confirmar classificação, argumentos, layout simétrico e ausência de ação de submissão.

**Acceptance Scenarios**:

1. **Given** uma notícia publicada classificada como refutada, **When** exibida, **Then** permanece acessível com classificação e argumentos.
2. **Given** uma alegação com evidências favoráveis e contrárias, **When** exibida, **Then** elas aparecem lado a lado, em simetria, com explicação neutra.
3. **Given** um usuário comum, **When** acessa a plataforma, **Then** visualiza notícias e evidências, sem opção de submeter.

---

### User Story 3 - Conferir fontes, atribuição e incerteza (Priority: P3)

O leitor confere as evidências (fonte, data e janela decisiva), entende quando o resultado se deve a incerteza ou conflito e sabe **a quem o veredito se refere**: se o veículo afirma diretamente, se relata a fala de alguém ("declaração de X") ou se atribui um dado a uma fonte ("a notícia atribui este dado a [fonte]").

**Why this priority**: confiança e auditabilidade; sustenta a promessa de não substituir o pensamento crítico.

**Independent Test**: abrir o detalhe de uma classificação e verificar fontes/datas/janela e o tipo de atribuição; verificar que casos sem evidência exibem incerteza.

**Acceptance Scenarios**:

1. **Given** uma classificação, **When** o leitor abre o detalhe, **Then** vê evidências com fonte, data e o trecho decisivo.
2. **Given** alegação do tipo declaração, **When** exibida, **Then** o resultado é apresentado como "Declaração de [X]".
3. **Given** alegação com dado atribuído a uma fonte, **When** exibida, **Then** o resultado informa que a notícia atribui o dado àquela fonte, permanecendo do veículo.

---

### User Story 4 - Revisor humano decide os casos pendentes (Priority: P3)

Um revisor designado pela plataforma recebe os casos sensíveis (refutada, conflitante envolvendo pessoa, ambígua, conflito numérico/ NLI, mudança de veredito entre versões), com as evidências e a diferença entre as versões do texto, e registra a decisão. Quem revisa (moderação, veículo ou ambos) é decisão da administração.

**Why this priority**: mantém o humano nas decisões de maior dano, sem a IA decidir sozinha sobre pessoas.

**Independent Test**: submeter notícia que gere caso refutado e confirmar que ele aparece na fila com evidências e diff, e que a publicação fica suspensa até a decisão.

**Acceptance Scenarios**:

1. **Given** uma alegação refutada, **When** a verificação termina, **Then** o caso entra na fila de revisão com evidências e status `pendente_revisao`.
2. **Given** duas versões da mesma notícia com mudança de veredito, **When** a nova versão é verificada, **Then** o revisor recebe a diferença entre as versões.
3. **Given** a decisão do revisor registrada, **Then** o status do caso é atualizado e o registro de auditoria é mantido.

---

### Edge Cases

- Notícia sem alegações verificáveis (opinião pura): informa que não há o que verificar, sem classificação.
- Alegação sobre evento futuro ou previsão: inconclusiva.
- Evidência publicada após o momento da submissão: não pode ser usada.
- Checagem de agência antiga para alegação sensível ao tempo: não é herdada; a alegação segue para busca própria.
- Notícia longa: até 10 alegações principais; o total de frases excluídas é registrado e exibido ao publicador.
- Alegação ambígua (substituição de referência com mais de um candidato): resultado limitado a inconclusiva até a revisão.
- Conteúdo em idioma diferente do português do Brasil: fora do escopo desta versão; o sistema informa.
- Falha na obtenção de evidências: declara incerteza, jamais "refutada".
- Fontes confiáveis divergentes: "conflitante" com motivo `agencias_divergem` (casamento) ou `evidencias_divergem` (veredito).
- Usuário comum (não verificado) tenta submeter: negado.
- Publicador com verificação revogada: novas submissões negadas.
- Notícia com caso pendente: não é publicada e o publicador é notificado.
- Ressubmissão que muda veredito de refutada para suportada/inconclusiva: exige revisão.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema DEVE permitir que apenas publicadores verificados submetam notícias para verificação.
- **FR-002**: O sistema DEVE selecionar as frases verificáveis da notícia (por regras: número, data, nome próprio ou verbo de declaração), limitadas a 10 alegações principais, registrando quantas ficaram de fora.
- **FR-003**: O sistema DEVE descontextualizar cada alegação e classificar sua atribuição (`fonte_citada`, `declaracao` ou `nenhuma`), resolvendo datas relativas a partir da data da notícia.
- **FR-004**: O sistema DEVE marcar se a alegação é sensível ao tempo (número, cargo ou índice sem data explícita).
- **FR-005**: O sistema DEVE casar a alegação com checagens existentes (similaridade + verificação bidirecional de equivalência + filtro temporal) e herdar o veredito humano quando houver casamento.
- **FR-006**: O sistema DEVE recuperar evidências externas por busca lexical e semântica com fusão, **nunca** usando a própria notícia como evidência.
- **FR-007**: O sistema DEVE classificar cada alegação em exatamente uma das quatro categorias: suportada, refutada, inconclusiva ou conflitante.
- **FR-008**: O sistema NÃO DEVE tratar ausência de evidência como falsidade.
- **FR-009**: O sistema DEVE apresentar, para cada classificação, as evidências com fonte, data e o trecho decisivo.
- **FR-010**: O sistema DEVE manter notícias refutadas visíveis após a publicação, sem bloqueá-las ou ocultá-las.
- **FR-011**: O sistema DEVE apresentar uma explicação em linguagem simples, ancorada nas evidências e neutra, identificando a quem o resultado se refere (veículo, declaração de X, ou dado atribuído à fonte).
- **FR-012**: O sistema DEVE considerar apenas evidências disponíveis até o momento da submissão, incluindo idade máxima para herança de checagens antigas em alegações sensíveis ao tempo.
- **FR-013**: O sistema DEVE declarar explicitamente incerteza ou conflito, com o motivo registrado.
- **FR-014**: O sistema DEVE operar em português do Brasil.
- **FR-015**: O sistema DEVE registrar, para auditoria, snapshots das bases, versão de parâmetros e modelos, substituições feitas e resultado de cada verificação.
- **FR-016**: O sistema DEVE negar a submissão de notícias por usuários não verificados.
- **FR-017**: O sistema DEVE permitir que usuários comuns visualizem as notícias publicadas, suas classificações e suas evidências.
- **FR-018**: O sistema DEVE distinguir os papéis de publicador verificado, usuário comum, revisor e administrador da plataforma.
- **FR-019**: O sistema DEVE apresentar, por alegação, as evidências favoráveis e contrárias de forma simétrica (lado a lado), com explicação neutra.
- **FR-020**: O sistema DEVE encaminhar à fila de revisão os casos sensíveis: toda refutada; conflitante envolvendo pessoa; alegação ambígua; conflito entre verificador numérico e o veredito; e mudança de veredito entre versões.
- **FR-021**: O sistema DEVE impedir a publicação enquanto houver caso pendente de revisão, notificando o publicador.
- **FR-022**: O sistema DEVE permitir ao publicador corrigir e ressubmeter a mesma notícia, até 3 versões, mantendo o histórico ligado.
- **FR-023**: O sistema DEVE detectar mudança de veredito entre versões da mesma notícia e encaminhá-la à revisão quando a mudança for de refutada para suportada ou inconclusiva.
- **FR-024**: O sistema NÃO DEVE enviar o texto não publicado a serviços externos no MVP.
- **FR-025**: O sistema DEVE manter o texto da notícia apenas durante a verificação e a revisão pendente, descartando-o após a publicação ou a conclusão sem pendências.

### Key Entities *(include if feature involves data)*

- **Publicador verificado**: veículo/organização autorizado pela administração; único papel que submete notícias.
- **Usuário comum**: leitor que visualiza e interage; não submete.
- **Revisor**: papel designado pela plataforma para decidir casos pendentes; mantém o registro da decisão.
- **Administrador da plataforma**: aprova/revoga publicadores (fora do MVP).
- **Notícia**: `noticia_id`, `versao_texto`, data; retenção transitória (não permanente).
- **Alegação**: frase verificável descontextualizada; tipo de atribuição, a quem se atribui, sensibilidade ao tempo e flag de ambiguidade.
- **Checagem de agência**: alegação já checada, veredito original convertido para as 4 classes, agência, data e link.
- **Casamento**: relação entre alegação e checagem, com probabilidade de equivalência e decisão de herança.
- **Trecho de evidência**: pedaço de documento confiável com fonte, domínio, URL e data de publicação.
- **Veredito**: classe final, origem (agência ou fluxo próprio), motivo, confiança (quando aplicável), status (final ou pendente de revisão) e trecho decisivo.
- **Explicação**: texto neutro por alegação, com fontes citadas; gerada por template no MVP.
- **Fila de revisão**: casos pendentes com evidências, diff entre versões e decisão do revisor.
- **Registro de auditoria**: snapshots das bases, versão de parâmetros/modelos, substituições e histórico de versões.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% das notícias submetidas com alegações verificáveis recebem uma das quatro classificações por alegação principal, nunca falsidade por falta de evidência.
- **SC-002**: Para 100% das classificações exibidas, o leitor consegue visualizar ao menos uma evidência com fonte e data.
- **SC-003**: A distinção entre "refutada" e "inconclusiva" é reportada separadamente e atinge desempenho igual ou superior às referências publicadas para o mesmo esquema de quatro classes (ex.: baselines do AVeriTeC e de Delucis et al., 2025).
- **SC-004**: 100% das notícias refutadas permanecem acessíveis após a publicação, com classificação e argumentos, sem ocultação.
- **SC-005**: Para uma notícia típica (≤1.000 palavras e ≤10 alegações), o resultado é emitido em até 2 minutos (p95), provisoriamente.
- **SC-006**: Em enquete pós-uso (escala de 1 a 5), ao menos 70% dos participantes concordam que a apresentação não entrega conclusão pronta e que formaram sua própria interpretação.
- **SC-007**: 100% das tentativas de submissão por usuários não verificados são negadas.
- **SC-008**: 100% dos casos sensíveis entram na fila de revisão antes de qualquer publicação.
- **SC-009**: 0% do texto não publicado é enviado a serviços externos no MVP.
- **SC-010**: 100% das ressubmissões mantêm histórico ligado e mudanças de veredito relevantes são sinalizadas ao revisor.

## Assumptions

- O público-alvo são leitores de notícias em geral, em português do Brasil, com acesso via navegador.
- O MVP verifica uma notícia por vez, em formato texto; imagem, áudio e vídeo estão fora do escopo.
- A submissão é restrita a publicadores verificados; aprovação/revogação é da administração e está fora do MVP (conjunto pré-aprovado).
- Interações sociais (comentar, compartilhar, reagir, criticar) pertencem à visão completa da TAKTA e estão fora deste MVP.
- No MVP, a base de evidência usa **fontes oficiais**; texto de veículos e agências só entra após parecer jurídico.
- O veredito **não** é decidido por LLM; a explicação usa templates no MVP, com LLM local como incremento.
- Texto não publicado não é enviado a serviços externos no MVP.
- Existe fila de revisão humana com um **revisor designado (papel único)** no MVP; quem revisa (moderação, veículo ou ambos) é decisão da administração (aberta no v3).
- No MVP, a notificação ao publicador é **passiva**: o status (inclusive pendência de revisão) é consultável no resultado; não há notificação ativa.
- Não há canal formal de contestação do publicador no MVP.
- O uso é acadêmico/experimental; restrições de licenciamento não comercial são aceitáveis; parecer jurídico é pré-requisito do piloto.
- A base pode não cobrir fatos muito recentes; nesse caso, a resposta esperada é "inconclusiva".
- Pendências de EDA: tamanhos/licença dos splits do AVERITEC-BR e existência de data por documento no knowledge store de avaliação.
