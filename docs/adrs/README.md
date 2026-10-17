# Architecture Decision Records

Seis decisões da reunião, em formato MADR adaptado. “Aceito” descreve a decisão histórica; não significa aprovação dos detalhes que a transcrição deixou em aberto nem implementação concluída.

| ADR | Assunto |
| --- | --- |
| [ADR-001](ADR-001-outbox-no-mysql.md) | Outbox no MySQL e atomicidade da mudança de status |
| [ADR-002](ADR-002-retry-com-backoff-e-dlq.md) | Retry limitado com backoff e DLQ persistida |
| [ADR-003](ADR-003-hmac-e-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint |
| [ADR-004](ADR-004-at-least-once-e-event-id.md) | At-least-once e deduplicação pelo consumidor |
| [ADR-005](ADR-005-worker-separado-com-polling.md) | Worker separado com polling de dois segundos |
| [ADR-006](ADR-006-reuso-dos-padroes-do-oms.md) | Reuso dos padrões do OMS |

Os nomes seguem o formato obrigatório ADR-NNN-titulo-em-kebab-case.md do [enunciado original](../ENUNCIADO.md), em substituição ao exemplo genérico anterior deste índice.

[RFC](../RFC.md): síntese arquitetural. [FDD](../FDD.md): detalhamento e lacunas. [TRACKER](../TRACKER.md): origem de cada registro.
