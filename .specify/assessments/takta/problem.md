# Problem Definition: leitores sem meio confiável de julgar notícias recém-publicadas

- **Slug**: takta
- **Created**: 2026-09-22
- **Inputs used**: intake.md? sim | research.md? sim | user input only? não

## Problem Statement

Leitores de notícias no Brasil não têm um meio rápido, transparente e baseado em evidências para julgar se as afirmações de uma notícia recém-publicada são factualmente sustentáveis. A verificação profissional é escassa e lenta, a desinformação circula antes de ser checada, e os detectores automáticos existentes em português são, em sua maioria, classificadores binários que inflam a acurácia por atalhos superficiais — sem separar "falso" de "não há evidência suficiente", o que trataría ausência de prova como falsidade.

## Affected Users & Stakeholders

- **Users**: leitores de notícias em geral (público brasileiro) — consomem conteúdo que pode ser desinformação e não dispõem de instrumento acessível para avaliar a confiabilidade factual antes de formar opinião. — [source: intake.md]
- **Users (produtores)**: publicadores verificados (veículos de comunicação e organizações editoriais) — submetem notícias para verificação antes de publicá-las; sua condição de verificado depende da administração da plataforma. — [source: intake.md (revisão 2026-09-27)]
- **Stakeholders**:
  - Grupo 5 da residência (Cauan, Isabel, Maria Gladys, Mickeias) — executores e avaliados.
  - Instituto Eldorado + UnB/FGA — instituições da residência; define critérios de sucesso e prazo.
  - Mentoria da residência — orienta escopo e articulações institucionais.
  - Administração da plataforma — aprova e revoga publicadores verificados; responsável pela governança dos papéis. — [source: revisão 2026-09-27]
  - Agências de checagem (Aos Fatos, Lupa, Comprova) — fontes de dados/labels e possíveis parceiras; já há caminho de articulação via Recod.ai/Horus. — [source: research.md, Relatório §7]
  - Veículos e organizações editoriais verificadas — publicadores na visão completa da TAKTA. — [source: intake.md]

## Goals

- Oferecer, para uma notícia submetida, uma classificação baseada em **evidências recuperadas** em quatro classes: suportada, refutada, inconclusiva, evidências conflitantes.
- Preservar a autonomia interpretativa do usuário: apresentar classificação + evidências, **sem entregar veredito seco**. — [source: intake.md]
- **Exibir notícias classificadas como desinformação junto com a classificação e os argumentos que a sustentam**, em vez de bloqueá-las — usando o próprio contraste como estímulo ao pensamento crítico. — [source: intake.md (revisão de 2026-09-22)]
- Declarar incerteza explicitamente quando não houver evidência suficiente, em vez de assumir falsidade. — [source: intake.md; Relatório §8]
- Demonstrar o fluxo ponta a ponta da FOP em uma interface web mínima (entrada: notícia; saída: classe + evidências).
- Restringir a submissão a publicadores verificados, mantendo usuários comuns como consumidores e interagentes. — [source: revisão 2026-09-27]

## Non-Goals

- Construir a plataforma social completa da TAKTA (páginas Notícias / Críticas / Populares). — [source: intake.md]
- Substituir jornalistas ou agências de checagem, ou emitir veredito binário verdadeiro/falso. — [source: intake.md; Relatório §3.2.3]
- **Bloquear ou ocultar notícias** com base na classificação — decisão revisada: o conteúdo permanece visível com sua classificação e evidências. — [source: intake.md (2026-09-22)]
- Implementar canal formal de contestação do publicador no MVP (aberto no v3). — [revisão 2026-09-27]
- A verificação avalia também a **fidelidade da citação** em alegações do tipo `declaracao` (aberto no v3). — [revisão 2026-09-27]
- Implementar cadastro, aprovação ou revogação de publicadores verificados (responsabilidade da administração da plataforma). — [source: revisão 2026-09-27]
- Implementar interações sociais (comentários, críticas, reações, vídeos) no MVP. — [source: revisão 2026-09-27]
- Verificação multimodal de imagem/vídeo (apesar de existir ImageFactCk.BR). — [source: Relatório §3.2.4]
- Exploração comercial (licença CC BY-NC do AVeriTeC impede uso lucrativo). — [source: Relatório §3.1.2]

## Success Metrics

- **Separar refutada de inconclusiva**, reportada isoladamente (medida que preserva a autonomia interpretativa). (baseline: desconhecida; AVeriTeC fornece baselines publicados) — [source: Relatório §5.2]
- **Desempenho na classe de evidências conflitantes** — categoria rara, de maior interesse público. (baseline: desconhecida) — [source: Relatório §5.2]
- **Taxa de abstenção calibrada** — proporção de casos com incerteza declarada confrontada com a taxa de erro observada. (baseline: desconhecida) — [source: Relatório §5.2]
- **Demonstração funcional do MVP**: interface web mínima processa uma notícia e retorna classe + evidências. (baseline: não existe) — [source: intake.md]
- **Estímulo ao pensamento crítico**: qualitativo — design que não entrega veredito seco; mensuração ainda indefinida. — [NEEDS CLARIFICATION: como medir] — [source: intake.md]

## Cost of Inaction

Se nada for construído, o grupo entrega apenas o levantamento documental: permanece a lacuna de um instrumento acessível de verificação baseada em evidências em PT-BR, e o debate sobre desinformação continua dependendo de checagem manual tardia. Além disso, perde-se a oportunidade, identificada no próprio levantamento, de construir o primeiro corpus nacional no formato AVeriTeC a partir do FactCenter. — [source: research.md; Relatório §6, §8]

## Open Questions

- [NEEDS CLARIFICATION: composição do híbrido generativo + não generativo (o que cada parte faz).]
- [NEEDS CLARIFICATION: como apresentar classificação e evidências sem enviesar a leitura nem amplificar a desinformação (ordem, destaque, linguagem).]
- [NEEDS CLARIFICATION: dataset prioritário para ajuste vs. avaliação no prazo da residência.]
- [NEEDS CLARIFICATION: LGPD, moderação de conteúdo e licenciamento não comercial.]
- [NEEDS CLARIFICATION: métrica de "estímulo ao pensamento crítico".]
- [NEEDS CLARIFICATION: critérios e fluxo de verificação de publicadores pela administração (fora do MVP; relevante para a plataforma completa).]
- [RESOLVIDO 2026-09-27] Revisão humana (fila para casos sensíveis), retenção transitória, apresentação simétrica, métrica de pensamento crítico e latência provisória foram definidas na spec/clarify; arquitetura definitiva em `fluxo-ia-fop-v3.md`.