# FOP — Requisitos de Usuário e de Sistema

**Projeto**: TAKTA / FOP — Residência em Inteligência Artificial e Análise de Dados (Instituto Eldorado + UnB/FGA) · Grupo 5
**Versão do documento**: 1.0 · **Data**: 2026-09-29

**Fontes de verdade**: `specs/001-fop-verificacao-alegacoes/spec.md` (requisitos de usuário e funcionais), `specs/001-fop-verificacao-alegacoes/plan.md` (não funcionais) e `fluxo-ia-fop-v3.md` (desenho do fluxo de IA).

**Escopo deste documento**: apenas **requisitos de usuário** (Parte I) e **requisitos de sistema** — funcionais e não funcionais (Parte II). Interfaces e matriz de rastreabilidade ficam para um documento posterior.

---

# Parte I — Requisitos de Usuário

## US1 — Publicador verificado submete e recebe a verificação antes de publicar (Prioridade: P1) 🎯 MVP

**Descrição**: Um publicador verificado submete uma notícia e recebe, por alegação, uma classificação em quatro categorias (suportada, refutada, inconclusiva, conflitante), com as evidências usadas (fonte, data e janela decisiva) e uma explicação neutra. Casos sensíveis ficam pendentes de revisão e a notícia não é publicada enquanto isso. O publicador pode corrigir e ressubmeter (até 3 versões).

**Por que esta prioridade**: é o núcleo do produto; sem esse fluxo a FOP não existe.

**Teste independente**: submeter uma notícia como publicador verificado e verificar que o resultado traz classes por alegação, evidências com fonte/data, explicação e status; e que notícia com caso sensível não publica até revisão.

**Cenários de aceite**:

1. **Dado** um publicador verificado, **quando** submete uma notícia, **então** recebe, por alegação, uma das quatro classificações com evidências (fonte/data) e explicação.
2. **Dado** alegações sem evidência suficiente, **quando** verificadas, **então** a classificação é "inconclusiva", nunca "refutada" por ausência de prova.
3. **Dado** evidências que apontam em direções opostas, **quando** verificadas, **então** a classificação é "conflitante", com o motivo registrado.
4. **Dada** uma alegação com veredito sensível (ex.: refutada), **quando** a verificação termina, **então** o caso vai para revisão e a notícia fica impedida de publicar até a decisão.
5. **Dado** um publicador que corrigiu a notícia, **quando** ressubmete (até 3 vezes), **então** uma nova versão é verificada e ligada ao histórico; mudança de refutada para suportada/inconclusiva exige revisão.

---

## US2 — Leitor vê notícias publicadas com classificação e evidências, sem bloqueio (Prioridade: P2)

**Descrição**: Usuários comuns visualizam as notícias publicadas (após a verificação e a revisão, quando houver) com a classificação por alegação; notícias refutadas permanecem visíveis com seus argumentos e evidências, em layout simétrico (favorável × contrária). O leitor não submete notícias.

**Por que esta prioridade**: materializa o princípio pedagógico e a política de conteúdo; sem isso o produto contradiz sua premissa.

**Teste independente**: acessar, como usuário comum, uma notícia publicada refutada e confirmar classificação, argumentos, layout simétrico e ausência de ação de submissão.

**Cenários de aceite**:

1. **Dada** uma notícia publicada classificada como refutada, **quando** exibida, **então** permanece acessível com classificação e argumentos.
2. **Dada** uma alegação com evidências favoráveis e contrárias, **quando** exibida, **então** elas aparecem lado a lado, em simetria, com explicação neutra.
3. **Dado** um usuário comum, **quando** acessa a plataforma, **então** visualiza notícias e evidências, sem opção de submeter.

---

## US3 — Conferir fontes, atribuição e incerteza (Prioridade: P3)

**Descrição**: O leitor confere as evidências (fonte, data e janela decisiva), entende quando o resultado se deve a incerteza ou conflito e sabe **a quem o veredito se refere**: se o veículo afirma diretamente, se relata a fala de alguém ("declaração de X") ou se atribui um dado a uma fonte ("a notícia atribui este dado a [fonte]").

**Por que esta prioridade**: confiança e auditabilidade; sustenta a promessa de não substituir o pensamento crítico.

**Teste independente**: abrir o detalhe de uma classificação e verificar fontes/datas/janela e o tipo de atribuição; verificar que casos sem evidência exibem incerteza.

**Cenários de aceite**:

1. **Dada** uma classificação, **quando** o leitor abre o detalhe, **então** vê evidências com fonte, data e o trecho decisivo.
2. **Dada** alegação do tipo declaração, **quando** exibida, **então** o resultado é apresentado como "Declaração de [X]".
3. **Dada** alegação com dado atribuído a uma fonte, **quando** exibida, **então** o resultado informa que a notícia atribui o dado àquela fonte, permanecendo do veículo.

---

## US4 — Revisor humano decide os casos pendentes (Prioridade: P3)

**Descrição**: Um revisor designado pela plataforma recebe os casos sensíveis (refutada, conflitante envolvendo pessoa, ambígua, conflito numérico/NLI, mudança de veredito entre versões), com as evidências e a diferença entre as versões do texto, e registra a decisão. Quem revisa (moderação, veículo ou ambos) é decisão da administração.

**Por que esta prioridade**: mantém o humano nas decisões de maior dano, sem a IA decidir sozinha sobre pessoas.

**Teste independente**: submeter notícia que gere caso refutado e confirmar que ele aparece na fila com evidências e diff, e que a publicação fica suspensa até a decisão.

**Cenários de aceite**:

1. **Dada** uma alegação refutada, **quando** a verificação termina, **então** o caso entra na fila de revisão com evidências e status `pendente_revisao`.
2. **Dadas** duas versões da mesma notícia com mudança de veredito, **quando** a nova versão é verificada, **então** o revisor recebe a diferença entre as versões.
3. **Dada** a decisão do revisor registrada, **então** o status do caso é atualizado e o registro de auditoria é mantido.

---

## Casos-limite (comportamento esperado do sistema)

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

---

# Parte II — Requisitos de Sistema

## 2.1 Requisitos Funcionais (FR)

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

## 2.2 Requisitos Não Funcionais (NFR)

| ID | Categoria | Requisito | Meta / verificação (MVP) |
|----|-----------|-----------|--------------------------|
| NFR-001 | Desempenho | Verificação assíncrona de notícia típica (≤1.000 palavras, ≤10 alegações) | p95 ≤120 s e p50 ≤60 s em CPU; publicador notificado ao concluir |
| NFR-002 | Desempenho | Orçamento por etapa (diagnóstico) | seleção ≤5 s; casamento ≤5 s; recuperação ≤20 s; veredito/NLI ≤60 s; calibração + explicação ≤15 s |
| NFR-003 | Escalabilidade | Concorrência e volume no MVP | 10 submissões simultâneas e ~50/dia sem degradar o p95 em >20%; fila assíncrona absorve pico de 10× |
| NFR-004 | Disponibilidade | Demo/avaliação | ≥95% na janela de avaliação; reinício automático; estado das verificações persistido localmente |
| NFR-005 | Confiabilidade | Falha de componente não é fatal | falha de recuperação/NLI/explicação → inconclusiva ou template; nenhum registro perdido; sem 5xx por dependência opcional |
| NFR-006 | Observabilidade | Log estruturado por verificação (JSON) | tempo por etapa, classe, abstenção, origem e versões (snapshots/config/modelos); métricas p50/p95, abstenção, fila de revisão; retenção de logs 90 dias |
| NFR-007 | Auditabilidade | Rastreabilidade e reprodutibilidade | veredito ligado a snapshots/config; mesmo snapshot + seed → mesmo resultado; registro imutável |
| NFR-008 | Segurança / Confidencialidade | Texto não publicado não sai para serviços externos no MVP | nenhuma chamada externa com o texto; segredos em variável de ambiente; acesso por papel; HTTPS |
| NFR-009 | Privacidade / LGPD | Minimização e retenção | texto transitório descartado após publicação/conclusão; logs sem conteúdo integral; exclusão sob solicitação; parecer jurídico antes do piloto |
| NFR-010 | Acessibilidade | Telas principais (submissão, revisão e leitura) | WCAG 2.1 AA: contraste ≥4,5:1, teclado, rótulos/ARIA; conteúdo em pt-BR |
| NFR-011 | Usabilidade / Transparência | Leitura compreensível e neutra | classificação e incerteza com fonte/data/janela; layout simétrico; atribuição explícita; leitura típica ≤3 min; nada bloqueado ao leitor |
| NFR-012 | Manutenibilidade | Qualidade de código e testes | cobertura ≥70% nos módulos críticos (veredito, filtros temporais, autorização, revisão); suíte ≤5 min em CPU; dependências fixadas; lint |
| NFR-013 | Portabilidade | Ambientes suportados | Linux/macOS/Windows + contêiner; execução **sem GPU**; demo em CPU |
| NFR-014 | Custo operacional | Orçamento do MVP | demo a custo ~zero (CPU + bibliotecas abertas); nenhuma API paga obrigatória; cache de alegações repetidas |

**Notas**: NFR-001/002 são provisórios até o benchmark de carga (E9) e o reajuste do piloto (E11). NFR-008 é atendido por não usar LLM por API no MVP (explicação por template; LLM local apenas como incremento).
