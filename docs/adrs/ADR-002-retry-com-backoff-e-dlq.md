# ADR-002 — Retry limitado com backoff e DLQ persistida

## Status

Aceito na reunião quanto à decisão central; detalhamento pendente onde indicado. Registro documental em 2026-10-16, elaborado com auxílio de IA e sujeito à revisão humana. A transcrição não informa a data civil da reunião. Não há implementação desta feature neste pacote.

## Contexto

**ADR-002-CTX.** Endpoints de clientes podem ficar indisponíveis, inclusive por manutenção de duas horas. A entrega precisa tolerar falhas sem manter tentativas automáticas indefinidamente.

Fonte: [09:15] Diego; [09:16] Diego.

## Decisão

**ADR-002-DEC.** Adotar a política literalmente fechada: “5 tentativas”, com backoff 1m/5m/30m/2h/12h, e depois DLQ em tabela separada. Persistir payload, motivo e timestamp. Replay manual recoloca o evento como pendente; exige ADMIN e auditoria de quem o executou.

Fonte: [09:17] Larissa; [09:18] Diego; [09:35] Diego; [09:36] Sofia; [09:36] Larissa; [09:48] Larissa.

## Detalhes associados

**ADR-002-DET.** A contagem é uma pendência bloqueadora do agendamento: cinco intervalos após uma primeira falha sugerem cinco retentativas, mas a reunião também afirma total de cinco tentativas. Não escolher entre cinco envios totais e inicial mais cinco retries. A soma dos intervalos citados é 14h36m; isso é cálculo sobre a sequência, não uma garantia operacional. A designação “exponencial” não autoriza inventar fórmula ou jitter.

Fonte: [09:17] Diego; [09:17] Larissa; [09:48] Larissa.

## Alternativas Consideradas

**ADR-002-ALT.** Três tentativas: rejeitadas por janela considerada curta. Retry indefinido: rejeitado por deixar eventos pendurados. Marcar falha apenas na outbox: preterido por tabela de DLQ separada, para facilitar leitura, investigação e reprocessamento.

Fonte: [09:15] Diego; [09:16] Bruno; [09:16] Diego; [09:17] Larissa; [09:18] Diego.

## Consequências

**ADR-002-CONS.** Positivas: falhas ficam persistidas e podem ser investigadas e reprocessadas; tentativas automáticas são limitadas. Negativas: eventos esgotados dependem de ação administrativa; há estado adicional para administrar. O trade-off é recuperação controlada em vez de esforço automático sem limite. Email foi explicitamente adiado. Replay em massa e preservação do event_id no replay não foram definidos.

Fonte: [09:15] Diego; [09:18] Diego; [09:36] Sofia; [09:37] Larissa.

## Questões em aberto

Antes de implementar o scheduler, resolver Q-01. Antes de operar, resolver sucesso/falha HTTP, recuperação após queda, passagem à DLQ e identidade no replay em Q-04/Q-08. Timeout de envio é 10s, conforme FDD; não confundir com a meta de latência do produto.

O [FDD](../FDD.md) concentra o registro de lacunas; o [RFC](../RFC.md) reúne os impactos arquiteturais. A origem de cada item está no [TRACKER](../TRACKER.md).

