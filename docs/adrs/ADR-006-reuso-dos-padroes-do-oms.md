# ADR-006 — Reuso dos padrões do OMS

## Status

Aceito na reunião quanto à decisão central; detalhamento pendente onde indicado. Registro documental em 2026-10-16, elaborado com auxílio de IA e sujeito à revisão humana. A transcrição não informa a data civil da reunião. Não há implementação desta feature neste pacote.

## Contexto

**ADR-006-CTX.** O OMS já organiza domínios em controllers, services, repositories, routes e schemas. Possui AppError, logger Pino, Zod e tratamento centralizado de erros.

Fonte: [09:27] Bruno; [09:28] Bruno; [09:29] Bruno.

## Decisão

**ADR-006-DEC.** Organizar webhooks como módulo seguindo esses padrões; reutilizar AppError, Pino, schemas Zod e middleware de erros. Adotar prefixo WEBHOOK_ para erros do módulo. Replay deve reutilizar requireRole com ADMIN.

Fonte: [09:29] Larissa; [09:30] Larissa; [09:36] Larissa.

## Detalhes associados

**ADR-006-DET.** A configuração de webhook usa autenticação normal para qualquer role nesta fase. customer_id deve ser informado no body ou path; a proposta inicial de extraí-lo do JWT foi corrigida durante a reunião.

Fonte: [09:32] Bruno; [09:32] Larissa; [09:37] Sofia.

## Alternativas Consideradas

**ADR-006-ALT.** Introduzir outro logger: rejeitado por Bruno ao afirmar que Pino já está no projeto e não será adicionado outro. Estrutura própria de camadas: alternativa analítica plausível ao reuso, não proposta nem votação registrada na reunião. Seu custo de divergência é avaliação deste ADR, não justificativa atribuída aos participantes.

Fonte: [09:27] Bruno; [09:29] Bruno; [09:30] Larissa.

## Consequências

**ADR-006-CONS.** Positivas: convenções e envelope de erros reconhecíveis; menor diversidade de infraestrutura. Negativas: decisões novas precisam caber nos limites reais dos componentes. O trade-off é consistência com adaptação explícita do domínio, em vez de assumir que os códigos existentes serão renomeados automaticamente.

Fonte: [09:28] Bruno; [09:29] Bruno; [09:30] Larissa.

## Integração existente

**ADR-006-COD.** [src/shared/errors/app-error.ts](../../src/shared/errors/app-error.ts) aceita status e código. Porém [src/middlewares/validate.middleware.ts](../../src/middlewares/validate.middleware.ts) produz VALIDATION_ERROR, e [src/middlewares/error.middleware.ts](../../src/middlewares/error.middleware.ts) mantém códigos genéricos. [src/middlewares/auth.middleware.ts](../../src/middlewares/auth.middleware.ts) fornece authenticate e requireRole. A composição ocorre em [src/app.ts](../../src/app.ts). Em [src/shared/errors/http-errors.ts](../../src/shared/errors/http-errors.ts), NotFoundError recebe apenas o recurso e fixa NOT_FOUND; ValidationError fixa VALIDATION_ERROR. O tratamento do processo worker não pertence à cadeia HTTP existente.

## Questões em aberto

Q-06 precisa delimitar códigos de domínio e erros compartilhados, sem afirmar que todo erro já será WEBHOOK_*. A revisão documental de 2026-10-17 detalha a integração em FDD-ERR-04: NotFoundError e ValidationError fixam códigos genéricos; erros específicos baseados em AppError e adaptação local de validação são **Proposta sujeita à aprovação**. O middleware Express não cobre automaticamente o worker separado. Essa explicação não altera a decisão histórica de reuso, nem aprova o alcance do prefixo. Q-09 continua aberta quanto à autorização por cliente e à atribuição confiável de ADMIN; o problema existente é dependência de segurança, sem correção de código.

O [FDD](../FDD.md) concentra o registro de lacunas; o [RFC](../RFC.md) reúne os impactos arquiteturais. A origem de cada item está no [TRACKER](../TRACKER.md).

