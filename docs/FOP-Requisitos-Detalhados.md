# FOP — Requisitos Detalhados (RF/RNF atômicos)

**Projeto**: TAKTA / FOP — Residência em IA (Instituto Eldorado + UnB/FGA) · Grupo 5
**Versão**: 1.0 · **Data**: 2026-09-29

**Natureza**: expansão **granular** (estilo SRS clássico) dos requisitos agregados do Spec Kit. Cada item é atômico e curto.
**Papel no handoff**: **anexo de verificação (checklist de aceitação)** — complementa a execução guiada por `spec.md`/`plan.md`/`contracts/`/`tasks.md`; **não os substitui**.
**Fontes**: `spec.md` (FR/SC), `plan.md` (NFR), `contracts/`, `data-model.md` e `fluxo-ia-fop-v3.md`.
**Rastreabilidade**: a coluna *Origem* liga cada requisito ao item agregado (FR/NFR do Spec Kit) e ao passo do v3. A visão consolidada permanece em `docs/FOP-Requisitos-Sistema-Usuario.md` e `docs/FOP-Matriz-Rastreabilidade.md`.

---

# Parte I — Requisitos Funcionais (RF)

## 1. Papéis e acesso

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-001 | O sistema deve permitir submissão de notícias apenas a publicadores verificados. | FR-001; v3 p1 |
| RF-002 | O sistema deve negar submissão de publicador desconhecido ou revogado (HTTP 403). | FR-016 |
| RF-003 | O sistema deve permitir a qualquer usuário visualizar notícias publicadas, classificações e evidências. | FR-017 |
| RF-004 | O sistema deve restringir a fila de revisão e a decisão ao papel de revisor. | FR-018 |
| RF-005 | O sistema deve distinguir os papéis publicador verificado, usuário comum, revisor e administrador. | FR-018 |
| RF-006 | O sistema deve conceder ao usuário comum apenas visualização e interação (sem submissão). | FR-017 |
| RF-007 | O sistema deve permitir ao administrador aprovar e revogar publicadores (fora do MVP). | FR-018; v3 §8.1 |

## 2. Submissão e versionamento

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-008 | O sistema deve receber título, texto e idioma da notícia na submissão. | contrato |
| RF-009 | O sistema deve aceitar notícias de até 1.000 palavras. | SC-005 |
| RF-010 | O sistema deve rejeitar texto vazio ou acima de 1.000 palavras (HTTP 422 `invalid_text`). | contrato |
| RF-011 | O sistema deve rejeitar conteúdo que não esteja em português do Brasil (HTTP 422 `unsupported_language`). | FR-014 |
| RF-012 | O sistema deve responder à submissão com HTTP 202 e identificadores (`noticia_id`, `versao_texto`). | contrato |
| RF-013 | O sistema deve criar a versão 1 na primeira submissão. | FR-022 |
| RF-014 | O sistema deve permitir ao publicador autor corrigir e ressubmeter a mesma notícia. | FR-022 |
| RF-015 | O sistema deve limitar a 3 versões por notícia. | FR-022; v3 C4 |
| RF-016 | O sistema deve responder HTTP 409 ao exceder o limite de versões (`max_versions_reached`). | contrato |
| RF-017 | O sistema deve ligar todas as versões ao mesmo `noticia_id`. | FR-022 |
| RF-018 | O sistema deve expor o histórico de versões (número, data, status, mudança de veredito). | FR-015; FR-022 |

## 3. Alegações (seleção, descontextualização e atribuição)

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-019 | O sistema deve selecionar frases verificáveis por sinais: número, data, nome próprio ou verbo de declaração. | FR-002; v3 p1 |
| RF-020 | O sistema deve limitar a verificação a 10 alegações principais por notícia. | FR-002; v3 p1 |
| RF-021 | O sistema deve registrar a quantidade de frases excluídas do corte. | FR-002; v3 W16 |
| RF-022 | O sistema deve exibir ao publicador a quantidade de frases excluídas. | FR-002; edge |
| RF-023 | O sistema deve descontextualizar cada alegação para que seja compreensível isoladamente. | FR-003; v3 p2 |
| RF-024 | O sistema deve substituir pronomes e referências apenas quando houver um único candidato. | FR-003; v3 p2 |
| RF-025 | O sistema deve marcar como ambígua a alegação que tiver mais de um candidato de referência. | FR-003; v3 p2 |
| RF-026 | O sistema deve limitar alegação ambígua à classificação inconclusiva até a revisão. | FR-003; FR-020 |
| RF-027 | O sistema deve classificar o tipo de atribuição: `fonte_citada`, `declaracao` ou `nenhuma`. | FR-003; v3 p2 |
| RF-028 | O sistema deve registrar a quem a declaração é atribuída (`atribuida_a`). | FR-003; v3 p2 |
| RF-029 | O sistema deve registrar a fonte citada quando a notícia atribui um dado a uma fonte. | FR-003; v3 p2 |
| RF-030 | O sistema deve converter datas relativas em absolutas usando a data da notícia. | FR-003; v3 p2 |
| RF-031 | O sistema deve marcar alegações sensíveis ao tempo (sem data explícita). | FR-004; v3 p2 C7 |

## 4. Casamento com checagens existentes

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-032 | O sistema deve buscar alegações já checadas por agências (top-k por similaridade). | FR-005; v3 p3 |
| RF-033 | O sistema deve exigir implicação bidirecional acima do limiar para confirmar casamento. | FR-005; v3 p3 |
| RF-034 | O sistema deve herdar o veredito da checagem quando houver casamento (origem `agencia`). | FR-005; v3 p3 |
| RF-035 | O sistema deve impedir herança de checagem mais antiga que a idade máxima em alegações sensíveis ao tempo. | FR-012; v3 p3 |
| RF-036 | O sistema deve marcar como conflitante (`agencias_divergem`) quando checagens casadas tiverem vereditos divergentes. | FR-013; v3 p3 |
| RF-037 | O sistema deve reaproveitar resultado em cache para alegação idêntica com os mesmos snapshots. | NFR-014; v3 Operação |
| RF-038 | O sistema deve invalidar o cache quando snapshots ou configuração mudarem. | NFR-014; v3 Operação |

## 5. Recuperação de evidência

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-039 | O sistema deve buscar trechos por palavras-chave (BM25) na base de evidência. | FR-006; v3 p4 |
| RF-040 | O sistema deve buscar trechos por significado (embeddings) na base de evidência. | FR-006; v3 p4 |
| RF-041 | O sistema deve fundir as duas listas de resultados (RRF). | FR-006; v3 p4 |
| RF-042 | O sistema deve descartar trechos provenientes da própria notícia ou de cópias dela. | FR-006; v3 p4 |
| RF-043 | O sistema deve descartar evidências publicadas após o momento da submissão. | FR-012; v3 p4 |
| RF-044 | O sistema deve priorizar trechos do domínio da fonte citada quando houver `fonte_citada`. | FR-003; v3 p4 |
| RF-045 | O sistema deve selecionar até 5 trechos para o veredito. | FR-006; v3 p4 |
| RF-046 | O sistema não deve usar a própria notícia como evidência. | FR-006 |

## 6. Veredito

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-047 | O sistema deve dividir cada trecho em janelas de 1 a 2 frases. | FR-007; v3 p5 |
| RF-048 | O sistema deve aplicar NLI entre a janela (premissa) e a alegação (hipótese). | FR-007; v3 p5 |
| RF-049 | O sistema deve registrar as maiores probabilidades de confirmação e contradição por trecho. | FR-007; v3 p5 |
| RF-050 | O sistema deve classificar trechos como fortes a favor ou contra pelos limiares configurados. | FR-007; v3 p5 |
| RF-051 | O sistema deve classificar como suportada a alegação com trechos fortes apenas a favor. | FR-007; v3 p5 |
| RF-052 | O sistema deve classificar como refutada a alegação com trechos fortes apenas contra. | FR-007; v3 p5 |
| RF-053 | O sistema deve classificar como conflitante a alegação com trechos fortes nos dois sentidos. | FR-007; v3 p5 |
| RF-054 | O sistema deve classificar como inconclusiva a alegação sem trecho forte. | FR-007; v3 p5 |
| RF-055 | O sistema deve registrar a janela decisiva que sustenta o veredito. | FR-009; v3 p5 |
| RF-056 | O sistema nunca deve classificar como refutada por ausência de evidência. | FR-008 |
| RF-057 | O sistema não deve atribuir confiança à classe inconclusiva. | FR-013; v3 p5 N4 |
| RF-058 | O sistema deve registrar o motivo do veredito quando aplicável. | FR-013 |

## 7. Calibração, abstenção e revisão

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-059 | O sistema deve calibrar a confiança das classes decisivas. | FR-007; v3 p6 |
| RF-060 | O sistema deve converter em inconclusiva (`baixa_confianca`) o veredito abaixo do limiar de abstenção. | FR-013; v3 p6 |
| RF-061 | O sistema deve encaminhar toda alegação refutada à fila de revisão. | FR-020; v3 p6 |
| RF-062 | O sistema deve encaminhar conflitante envolvendo pessoa à fila de revisão. | FR-020; v3 p6 |
| RF-063 | O sistema deve encaminhar alegação ambígua à fila de revisão. | FR-020; v3 p6 |
| RF-064 | O sistema deve encaminhar conflito entre verificador numérico e NLI à fila de revisão. | FR-020; v3 p6 |
| RF-065 | O sistema deve encaminhar mudança de veredito entre versões à fila de revisão. | FR-023; v3 C4 |
| RF-066 | O sistema deve impedir a publicação enquanto houver caso pendente de revisão. | FR-021 |
| RF-067 | O sistema deve expor ao publicador o status de pendência (consulta). | FR-021; premissa MVP |
| RF-068 | O sistema deve permitir ao revisor decidir (aceitar, rejeitar, ajustar) com justificativa. | FR-020; contrato |
| RF-069 | O sistema deve registrar a decisão do revisor (identidade, data e justificativa). | FR-015; v3 p6 |
| RF-070 | O sistema deve fornecer ao revisor a diferença entre as versões do texto. | FR-023; v3 C4 |
| RF-071 | O sistema deve listar a fila de revisão por status, com motivo e evidências. | FR-020; contrato |

## 8. Explicação e exibição

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-072 | O sistema deve gerar explicação em linguagem simples e neutra por alegação. | FR-011 |
| RF-073 | O sistema deve citar as fontes usadas na explicação. | FR-011 |
| RF-074 | O sistema deve prefixar "Declaração de [X]:" em alegações do tipo declaração. | FR-011; v3 p7 |
| RF-075 | O sistema deve prefixar "A notícia atribui este dado a [fonte]:" em alegações com fonte citada. | FR-011; v3 p7 |
| RF-076 | O sistema deve limitar a janela decisiva exibida a 25 palavras. | FR-009; v3 p7 |
| RF-077 | O sistema deve exibir as evidências com fonte e data. | FR-009 |
| RF-078 | O sistema deve apresentar evidências favoráveis e contrárias lado a lado (simetria). | FR-019 |
| RF-079 | O sistema deve manter notícias refutadas visíveis, com classificação e argumentos. | FR-010 |
| RF-080 | O sistema não deve ocultar nem bloquear notícias em função da classe. | FR-010 |
| RF-081 | O sistema deve exibir incerteza ou conflito com o motivo registrado. | FR-013 |
| RF-082 | O sistema deve informar quando a notícia não tiver alegação verificável. | edge |
| RF-083 | O sistema deve classificar alegações sobre eventos futuros como inconclusivas. | edge |
| RF-084 | O sistema deve informar a limitação quando o conteúdo não estiver em pt-BR. | FR-014; edge |
| RF-085 | O sistema deve operar todo o fluxo em português do Brasil. | FR-014 |

## 9. Auditoria, retenção e confidencialidade

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-086 | O sistema deve registrar por veredito: snapshot P1, snapshot P2, versão de config e modelos. | FR-015 |
| RF-087 | O sistema deve registrar as substituições de referência realizadas. | FR-015; v3 p6.4 |
| RF-088 | O sistema deve manter o registro de auditoria imutável. | FR-015; v3 p6.4 |
| RF-089 | O sistema deve expor as versões de snapshots no endpoint de saúde. | FR-015; contrato |
| RF-090 | O sistema não deve persistir o texto integral da notícia após a conclusão. | FR-025 |
| RF-091 | O sistema deve manter o texto apenas durante a verificação e a revisão pendente. | FR-025 |
| RF-092 | O sistema deve descartar o texto após a publicação ou conclusão sem pendências. | FR-025 |
| RF-093 | O sistema não deve enviar texto não publicado a serviços externos. | FR-024 |
| RF-094 | O sistema deve ler segredos apenas de variáveis de ambiente. | NFR-008 |
| RF-095 | O sistema deve expor no resultado a política de não bloqueio e de retenção transitória. | FR-010; FR-025 |

## 10. Preparação das bases (P1–P3)

| ID | Requisito | Origem |
|----|-----------|--------|
| RF-096 | O sistema deve construir a base de checagens de agências com veredito mapeado para as 4 classes. | v3 P1 |
| RF-097 | O sistema deve deduplicar checagens e agrupar a mesma alegação verificada por várias agências. | v3 P1 |
| RF-098 | O sistema deve marcar como conflitante (`agencias_divergem`) grupos com vereditos divergentes. | v3 P1 |
| RF-099 | O sistema deve construir a base de evidência a partir de fontes oficiais. | v3 P2; premissa |
| RF-100 | O sistema deve cortar documentos em trechos de ~150 palavras com sobreposição de ~30. | v3 P2 |
| RF-101 | O sistema deve guardar fonte, domínio, URL e data de publicação de cada trecho. | v3 P2 |
| RF-102 | O sistema deve indexar a base de evidência por BM25 e por embeddings. | v3 P2 |
| RF-103 | O sistema deve registrar um snapshot ao final de cada atualização das bases. | v3 P1/P2 |
| RF-104 | O sistema deve escolher os limiares no train-A do AVERITEC-BR. | v3 P3 |
| RF-105 | O sistema deve ajustar os calibradores no train-B com seeds fixas. | v3 P3 |
| RF-106 | O sistema deve exigir reindexação quando o modelo de embeddings for trocado. | v3 P1/P2 |

---

# Parte II — Requisitos Não Funcionais (RNF)

## 11. Desempenho e capacidade

| ID | Requisito | Origem |
|----|-----------|--------|
| RNF-001 | O sistema deve responder à verificação de notícia típica (≤1.000 palavras; ≤10 alegações) em p95 ≤120 s e p50 ≤60 s. | NFR-001; SC-005 |
| RNF-002 | O sistema deve respeitar o orçamento por etapa: seleção ≤5 s; casamento ≤5 s; recuperação ≤20 s; veredito ≤60 s; calibração+explicação ≤15 s. | NFR-002 |
| RNF-003 | O sistema deve suportar 10 submissões simultâneas. | NFR-003 |
| RNF-004 | O sistema deve suportar ~50 notícias por dia. | NFR-003 |
| RNF-005 | O sistema deve absorver pico de 10× o volume médio via fila assíncrona. | NFR-003; E9 |

## 12. Disponibilidade e confiabilidade

| ID | Requisito | Origem |
|----|-----------|--------|
| RNF-006 | O sistema deve manter ≥95% de disponibilidade na janela de avaliação. | NFR-004 |
| RNF-007 | O sistema deve reiniciar automaticamente e preservar o estado das verificações. | NFR-004 |
| RNF-008 | O sistema deve degradar para inconclusiva ou template em falha de componente, sem perder registros. | NFR-005 |
| RNF-009 | O sistema não deve retornar 5xx por falha de dependência opcional. | NFR-005 |

## 13. Observabilidade e auditabilidade

| ID | Requisito | Origem |
|----|-----------|--------|
| RNF-010 | O sistema deve gerar log estruturado (JSON) por verificação com tempos, classe, origem e versões. | NFR-006 |
| RNF-011 | O sistema deve expor métricas de p50/p95, taxa de abstenção e tamanho da fila de revisão. | NFR-006 |
| RNF-012 | O sistema deve reter logs por 90 dias. | NFR-006 |
| RNF-013 | O sistema deve reproduzir o mesmo resultado com o mesmo snapshot e seed. | NFR-007 |

## 14. Segurança, privacidade e conformidade

| ID | Requisito | Origem |
|----|-----------|--------|
| RNF-014 | O sistema deve restringir o acesso por papel e usar HTTPS. | NFR-008 |
| RNF-015 | O sistema deve garantir 0% do texto não publicado enviado a terceiros. | NFR-008; SC-009 |
| RNF-016 | O sistema não deve registrar conteúdo integral da notícia em logs. | NFR-009 |
| RNF-017 | O sistema deve permitir exclusão de registros sob solicitação. | NFR-009 |
| RNF-018 | O sistema deve exigir parecer jurídico antes do piloto (LGPD, direito autoral, reuso de vereditos). | NFR-009; v3 §8.7 |

## 15. Usabilidade, acessibilidade e qualidade

| ID | Requisito | Origem |
|----|-----------|--------|
| RNF-019 | O sistema deve atender WCAG 2.1 AA nas telas principais. | NFR-010 |
| RNF-020 | O sistema deve permitir leitura típica em até 3 minutos. | NFR-011 |
| RNF-021 | O sistema deve manter cobertura de testes ≥70% nos módulos críticos. | NFR-012 |
| RNF-022 | O sistema deve executar a suíte de testes em ≤5 minutos em CPU. | NFR-012 |
| RNF-023 | O sistema deve manter dependências fixadas e lint configurado. | NFR-012 |
| RNF-024 | O sistema deve rodar em Linux, macOS, Windows e contêiner. | NFR-013 |
| RNF-025 | O sistema deve executar sem GPU (demo em CPU). | NFR-013 |
| RNF-026 | O sistema deve operar a custo ~zero no MVP, sem API paga obrigatória. | NFR-014 |

## 16. Notas de verificação

- **RNF-001/RNF-002** são provisórios até o benchmark de carga (E9) e o reajuste do piloto (E11).
- **RNF-006/RNF-007** são verificados no deploy da demo (smoke test).
- **RNF-021/RNF-022** são verificados por `pytest --cov` com gate em CI.
