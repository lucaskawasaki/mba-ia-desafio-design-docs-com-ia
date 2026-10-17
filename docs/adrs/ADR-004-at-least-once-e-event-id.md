# ADR-004 — At-least-once e deduplicação pelo consumidor

## Status

Aceito na reunião quanto à decisão central; detalhamento pendente onde indicado. Registro documental em 2026-10-16, elaborado com auxílio de IA e sujeito à revisão humana. A transcrição não informa a data civil da reunião. Não há implementação desta feature neste pacote.

## Contexto

**ADR-004-CTX.** A entrega HTTP pode produzir duplicatas. O consumidor precisa reconhecer repetições do mesmo evento sem tratar cada chamada como uma mudança distinta.

Fonte: [09:24] Diego; [09:25] Bruno; [09:25] Diego.

## Decisão

**ADR-004-DEC.** Adotar at-least-once com event_id UUID gerado na inserção da outbox e enviado em X-Event-Id. O cliente deduplica pelo identificador. A repetição de envio do mesmo evento usa sua identidade persistida; a identidade do replay administrativo ainda precisa de definição.

Fonte: [09:25] Diego; [09:26] Larissa.

## Detalhes associados

**ADR-004-DET.** Marcos se comprometeu a destacar a deduplicação no portal dos desenvolvedores. Isso é encaminhamento de comunicação, não evidência de portal atualizado.

Fonte: [09:26] Marcos.

## Alternativas Consideradas

**ADR-004-ALT.** Exactly-once: descartado pela coordenação adicional entre ambos os lados e maior complexidade. A expressão “resolve 99% dos casos” usada na discussão não constitui SLO ou métrica de sucesso.

Fonte: [09:25] Diego.

## Consequências

**ADR-004-CONS.** Positivas: identificador estável permite reconhecer tentativas repetidas. Negativas: efeitos idempotentes dependem do consumidor; duplicatas são esperadas pelo contrato. O trade-off é simplicidade de entrega com responsabilidade de deduplicação do receptor. Com retry finito e DLQ, não há promessa de sucesso remoto para endpoint permanentemente indisponível; preserva-se a intenção de entrega e a recuperação manual.

Fonte: [09:25] Diego; [09:25] Sofia; [09:17] Larissa; [09:18] Diego.

## Questões em aberto

Q-08 deve resolver se replay preserva a identidade e como múltiplos endpoints compartilham ou não event_id. Não confundir identificação do evento, identificação do endpoint e ordenação. Ordering fica no ADR-005.

O [FDD](../FDD.md) concentra o registro de lacunas; o [RFC](../RFC.md) reúne os impactos arquiteturais. A origem de cada item está no [TRACKER](../TRACKER.md).

