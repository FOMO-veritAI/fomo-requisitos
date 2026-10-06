# TAKTA / FOP Constitution

## Core Principles

### I. Autonomia do Leitor (NÃO NEGOCIÁVEL)

A FOP existe para apoiar o julgamento humano, nunca para substituí-lo. Toda classificação MUST
ser apresentada com as evidências que a sustentam e MUST NOT prescrever uma conclusão pronta.
O veredito MUST ser produzido por componentes auditáveis (regras + NLI calibrado); LLM MUST NOT
decidir a classe. Explicações MUST ser neutras e identificar a quem o resultado se refere
(veículo, declaração de terceiro ou dado atribuído a fonte).

Rationale: o Challenge 1 e a proposta da TAKTA definem que a IA combate a desinformação sem
retirar do usuário sua autonomia de pensamento.

### II. Incerteza Honesta

Ausência de evidência MUST NOT ser tratada como falsidade. Quando não houver evidência
suficiente, o resultado MUST ser "inconclusiva"; quando fontes confiáveis divergirem, MUST ser
"evidências conflitantes". A abstenção MUST ser calibrada e registrada, e a classe inconclusiva
MUST NOT receber pontuação de confiança.

Rationale: distinguir "falso" de "não verificado" é o que preserva a autonomia interpretativa e
evita condenações injustas.

### III. Evidência Rastreável e Auditável

Toda classificação MUST citar evidência externa com fonte, data e trecho decisivo, e MUST NOT
usar a própria notícia como evidência. Apenas evidências disponíveis até o momento da submissão
podem ser usadas. Cada veredito MUST ser rastreável a snapshots das bases, versão de parâmetros e
modelos, e o registro de auditoria MUST ser imutável.

Rationale: sem rastreabilidade não há verificação defensável nem reprodutibilidade científica.

### IV. Confidencialidade e Minimização de Dados

Texto não publicado MUST NOT ser enviado a serviços externos. Segredos MUST ficar apenas em
variáveis de ambiente. O texto da notícia MUST ter retenção transitória, mantido somente durante
a verificação e a revisão pendente e descartado após a publicação ou conclusão. O sistema MUST
operar em português do Brasil.

Rationale: protege o veículo e o processo editorial (LGPD e confidencialidade pré-publicação).

### V. Humano nas Decisões de Dano

Casos sensíveis MUST passar por revisão humana antes de qualquer publicação: toda refutada;
conflitante envolvendo pessoa; alegação ambígua; conflito entre verificador numérico e o
veredito; e mudança de veredito entre versões. Enquanto houver caso pendente, a publicação MUST
ficar suspensa e o publicador MUST ser notificado (status consultável no MVP).

Rationale: mantém o humano nas decisões de maior dano e evita punir notícias sem revisão.

## Restrições Adicionais

- A submissão de notícias MUST ser restrita a publicadores verificados (veículos/organizações
  aprovados pela administração); usuários comuns apenas visualizam e interagem.
- Notícias refutadas MUST permanecer visíveis após a publicação, com classificação e argumentos
  (sem bloqueio ou ocultação).
- No MVP, a base de evidência MUST usar fontes oficiais; texto de veículos/agências só entra após
  parecer jurídico.
- O uso MUST ser acadêmico e não comercial (licenças como CC BY-NC do AVeriTeC); parecer jurídico
  (LGPD, direito autoral, reuso de vereditos) é pré-requisito do piloto.
- A interface MUST apresentar evidências favoráveis e contrárias de forma simétrica.

## Fluxo de Desenvolvimento e Qualidade

- O projeto segue Spec-Driven Development: `spec → plan → tasks → analyze` antes de implementar;
  mudanças de escopo MUST ser refletidas na spec e no plano.
- As especificações técnicas herdadas do `fluxo-ia-fop-v3.md` são vinculantes; desvios MUST ser
  justificados por escrito e registrados.
- Testes MUST cobrir os módulos críticos (veredito, filtros temporais, autorização, revisão) com
  cobertura ≥70%; a avaliação MUST reportar métricas por classe (refutada × inconclusiva),
  calibração e abstenção, com verificação anti-vazamento.
- As métricas de sucesso (SC-001–SC-010) e os NFRs do `plan.md` MUST ser verificados a cada
  incremento.

## Governance

- A constituição supersede as demais práticas; conflitos MUST ser resolvidos ajustando spec,
  plano ou tarefas — nunca diluindo um princípio.
- Emendas MUST ser documentadas, aprovadas pelo grupo e refletidas na versão semântica: MAJOR
  para remoção/redefinição incompatível, MINOR para novo princípio/seção, PATCH para
  esclarecimentos.
- Toda revisão de PR/entrega MUST verificar conformidade com os princípios; desvios exigem
  justificativa registrada.
- Datas e versões seguem ISO 8601 (YYYY-MM-DD) e SemVer.

**Version**: 1.0.0 | **Ratified**: 2026-09-29 | **Last Amended**: 2026-09-29
