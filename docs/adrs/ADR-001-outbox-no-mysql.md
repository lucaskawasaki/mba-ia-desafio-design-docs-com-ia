# ADR-001 — Outbox no MySQL e atomicidade da mudança de status

## Status

Aceito na reunião quanto à decisão central; detalhamento pendente onde indicado. Registro documental em 2026-10-16, elaborado com auxílio de IA e sujeito à revisão humana. A transcrição não informa a data civil da reunião. Não há implementação desta feature neste pacote.

## Contexto

**ADR-001-CTX.** Os clientes consultam pedidos repetidamente para detectar mudanças. Fazer HTTP na transação prenderia a mudança de status ao tempo de resposta de terceiros.

Fonte: [09:00] Marcos; [09:04] Bruno.

## Decisão

**ADR-001-DEC.** Registrar o evento elegível na outbox do MySQL na mesma transação que altera pedido e histórico. Se a inserção falhar, a mudança deve sofrer rollback. A entrega HTTP acontece depois, pelo worker separado. A ausência de endpoints interessados no status dispensa inserção; atomicidade não significa criar eventos sem assinantes.

Fonte: [09:06] Diego; [09:08] Larissa; [09:34] Bruno; [09:40] Bruno; [09:41] Diego.

## Detalhes associados

**ADR-001-DET.** Usar UUID na outbox e persistir o payload renderizado na inserção, preservando o estado da mudança. Esses detalhes foram fechados após o resumo principal, às 09:51 e 09:52.

Fonte: [09:51] Larissa; [09:52] Larissa; [09:52] Diego; [09:52] Bruno.

## Alternativas Consideradas

**ADR-001-ALT.** HTTP síncrono: descartado por bloquear a transação e acoplar sua disponibilidade ao cliente. Redis Streams: descartado pela infraestrutura adicional para um time pequeno. Renderizar somente no envio: rejeitado porque o pedido pode ter mudado. ID autoincremental: preterido por UUID.

Fonte: [09:04] Bruno; [09:07] Larissa; [09:07] Diego; [09:51] Diego; [09:51] Larissa; [09:52] Larissa.

## Consequências

**ADR-001-CONS.** Positivas: estado do pedido e registro do evento compartilham commit/rollback; snapshot evita consultar um estado posterior. Negativas: mais trabalho e possibilidade de falha na transação principal; crescimento da outbox no mesmo banco. O trade-off é consistência transacional com carga adicional no MySQL, em vez de infraestrutura de mensageria separada. Esta análise não garante recebimento remoto; a política de falhas está no ADR-002.

Fonte: [09:06] Diego; [09:07] Bruno; [09:08] Diego; [09:40] Bruno; [09:52] Larissa.

## Integração existente

**ADR-001-COD.** O ponto existente é OrderService.changeStatus() em [src/modules/orders/order.service.ts](../../src/modules/orders/order.service.ts), usando Prisma.TransactionClient. [prisma/schema.prisma](../../prisma/schema.prisma) ainda não tem entidades de webhook. OrderNumberSequence usa ID inteiro: a fala “tudo é uuid” não descreve literalmente todos os modelos.

## Questões em aberto

Granularidade por endpoint, índices físicos, recuperação e efeito de payload excessivo ainda precisam de desenho. A interface publishWebhookEvent(tx, order, fromStatus, toStatus) é proposta apoiada em [09:41] Bruno e [09:41] Diego, não código existente. Ver Q-04, Q-07 e Q-10 no FDD.

O [FDD](../FDD.md) concentra o registro de lacunas; o [RFC](../RFC.md) reúne os impactos arquiteturais. A origem de cada item está no [TRACKER](../TRACKER.md).

