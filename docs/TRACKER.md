# Tracker de rastreabilidade

## Método e limites

Este registro liga os itens identificáveis do PRD, RFC, FDD e dos seis ADRs às fontes locais: [TRANSCRICAO.md](../TRANSCRICAO.md) e código do commit e7f6311b96ee3b21bf8cb7077f53a6e4045143eb.

- Cada ID corresponde a uma unidade identificável marcada no documento. Menções posteriores ao mesmo ID são referências, não novos requisitos.
- A coluna Tipo distingue requisito/decisão de alternativa descartada, adiamento, evidência do código, lacuna e proposta derivada.
- Para propostas e lacunas, a fonte sustenta o problema ou a convenção de origem. **Ela não significa que os participantes aprovaram o preenchimento proposto.**
- Exemplos fictícios de valores e schemas pertencem à unidade de contrato identificada no FDD. Os status HTTP, envelopes e paths propostos também dependem de FDD-CONV-01/FDD-INT-10, além da fonte funcional de cada operação.
- Consequências e testes derivados não são novas decisões históricas. Sua identificação deixa explícita essa diferença.
- Requisitos editoriais do desafio e fatos do processo são auditados contra [ENUNCIADO.md](ENUNCIADO.md) em [AUDITORIA.md](AUDITORIA.md). Não são classificados artificialmente como código ou fala de reunião.

Revisão documental: 2026-10-17. As probabilidades qualitativas do PRD e a classificação de implementabilidade do FDD são avaliações autorais, não falas dos participantes. FDD-EX-01, FDD-EX-02 e FDD-ERR-04 identificam os detalhamentos acrescentados nesta revisão.

## Cobertura registrada

151 unidades identificadas; 128 linhas de TRANSCRICAO (84,77%) e 23 de CODIGO. Há uma linha por unidade, sem duplicar a mesma unidade para inflar a proporção de fontes. A verificação de correspondência entre IDs e documentos, fontes e caminhos é descrita na auditoria.

Cobertura das unidades com ID: 151/151 (100%). Esse número não substitui a revisão semântica dos parágrafos: todo requisito, decisão ou restrição adicional sem ID deve ser vinculado a uma unidade pertinente ou incluído no denominador. Cabeçalhos, metadados, navegação e fatos do processo não são requisitos da feature.

## Tabela de rastreabilidade

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-001-CTX | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | Os clientes consultam pedidos repetidamente para detectar mudanças | TRANSCRICAO | [09:00] Marcos; [09:04] Bruno |
| ADR-001-DEC | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Registrar o evento elegível na outbox do MySQL na mesma transação que altera pedido e histórico | TRANSCRICAO | [09:06] Diego; [09:08] Larissa; [09:34] Bruno; [09:40] Bruno; [09:41] Diego |
| ADR-001-DET | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão / restrição | Usar UUID na outbox e persistir o payload renderizado na inserção, preservando o estado da mudança | TRANSCRICAO | [09:51] Larissa; [09:52] Larissa; [09:52] Diego; [09:52] Bruno |
| ADR-001-ALT | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa / trade-off | HTTP síncrono: descartado por bloquear a transação e acoplar sua disponibilidade ao cliente | TRANSCRICAO | [09:04] Bruno; [09:07] Larissa; [09:07] Diego; [09:51] Diego; [09:51] Larissa; [09:52] Larissa |
| ADR-001-CONS | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência derivada | Positivas: estado do pedido e registro do evento compartilham commit/rollback; snapshot evita consultar um estado posterior | TRANSCRICAO | [09:06] Diego; [09:07] Bruno; [09:08] Diego; [09:40] Bruno; [09:52] Larissa |
| ADR-001-COD | docs/adrs/ADR-001-outbox-no-mysql.md | Evidência do código | Pontos existentes e limites de integração | CODIGO | src/modules/orders/order.service.ts; prisma/schema.prisma |
| ADR-002-CTX | docs/adrs/ADR-002-retry-com-backoff-e-dlq.md | Contexto | Endpoints de clientes podem ficar indisponíveis, inclusive por manutenção de duas horas | TRANSCRICAO | [09:15] Diego; [09:16] Diego |
| ADR-002-DEC | docs/adrs/ADR-002-retry-com-backoff-e-dlq.md | Decisão | Adotar a política literalmente fechada: “5 tentativas”, com backoff 1m/5m/30m/2h/12h, e depois DLQ em tabela separada | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:35] Diego; [09:36] Sofia; [09:36] Larissa; [09:48] Larissa |
| ADR-002-DET | docs/adrs/ADR-002-retry-com-backoff-e-dlq.md | Lacuna identificada | A contagem é uma pendência bloqueadora do agendamento: cinco intervalos após uma primeira falha sugerem cinco retentativas, mas a reunião também afirma total de cinco tentativas | TRANSCRICAO | [09:17] Diego; [09:17] Larissa; [09:48] Larissa |
| ADR-002-ALT | docs/adrs/ADR-002-retry-com-backoff-e-dlq.md | Alternativa / trade-off | Três tentativas: rejeitadas por janela considerada curta | TRANSCRICAO | [09:15] Diego; [09:16] Bruno; [09:16] Diego; [09:17] Larissa; [09:18] Diego |
| ADR-002-CONS | docs/adrs/ADR-002-retry-com-backoff-e-dlq.md | Consequência derivada | Positivas: falhas ficam persistidas e podem ser investigadas e reprocessadas; tentativas automáticas são limitadas | TRANSCRICAO | [09:15] Diego; [09:18] Diego; [09:36] Sofia; [09:37] Larissa |
| ADR-003-CTX | docs/adrs/ADR-003-hmac-e-secret-por-endpoint.md | Contexto | O receptor precisa verificar a origem e integridade dos dados enviados para fora da infraestrutura do OMS | TRANSCRICAO | [09:19] Sofia; [09:22] Diego |
| ADR-003-DEC | docs/adrs/ADR-003-hmac-e-secret-por-endpoint.md | Decisão | Assinar o corpo da requisição com HMAC-SHA256, usando secret exclusiva por endpoint | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:31] Marcos |
| ADR-003-DET | docs/adrs/ADR-003-hmac-e-secret-por-endpoint.md | Decisão / restrição | HTTPS é obrigatório | TRANSCRICAO | [09:23] Sofia; [09:44] Diego; [09:46] Sofia; [09:47] Larissa |
| ADR-003-ALT | docs/adrs/ADR-003-hmac-e-secret-por-endpoint.md | Alternativa / trade-off | Secret global: explicitamente rejeitada porque o vazamento afetaria todos os endpoints | TRANSCRICAO | [09:21] Sofia |
| ADR-003-CONS | docs/adrs/ADR-003-hmac-e-secret-por-endpoint.md | Consequência derivada | Positivas: comprometimento de uma secret não expõe automaticamente todas as configurações; a rotação prevê tempo de migração | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| ADR-004-CTX | docs/adrs/ADR-004-at-least-once-e-event-id.md | Contexto | A entrega HTTP pode produzir duplicatas | TRANSCRICAO | [09:24] Diego; [09:25] Bruno; [09:25] Diego |
| ADR-004-DEC | docs/adrs/ADR-004-at-least-once-e-event-id.md | Decisão | Adotar at-least-once com event_id UUID gerado na inserção da outbox e enviado em X-Event-Id | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| ADR-004-DET | docs/adrs/ADR-004-at-least-once-e-event-id.md | Decisão / restrição | Marcos se comprometeu a destacar a deduplicação no portal dos desenvolvedores | TRANSCRICAO | [09:26] Marcos |
| ADR-004-ALT | docs/adrs/ADR-004-at-least-once-e-event-id.md | Alternativa / trade-off | Exactly-once: descartado pela coordenação adicional entre ambos os lados e maior complexidade | TRANSCRICAO | [09:25] Diego |
| ADR-004-CONS | docs/adrs/ADR-004-at-least-once-e-event-id.md | Consequência derivada | Positivas: identificador estável permite reconhecer tentativas repetidas | TRANSCRICAO | [09:25] Diego; [09:25] Sofia; [09:17] Larissa; [09:18] Diego |
| ADR-005-CTX | docs/adrs/ADR-005-worker-separado-com-polling.md | Contexto | A entrega deve ser assíncrona e atender à necessidade de notificações abaixo de 10 segundos | TRANSCRICAO | [09:02] Marcos; [09:09] Diego |
| ADR-005-DEC | docs/adrs/ADR-005-worker-separado-com-polling.md | Decisão | Executar worker em processo separado, com polling de 2 segundos dos eventos pendentes mais antigos | TRANSCRICAO | [09:09] Diego; [09:10] Larissa; [09:11] Diego; [09:12] Diego; [09:30] Bruno |
| ADR-005-DET | docs/adrs/ADR-005-worker-separado-com-polling.md | Decisão / restrição | A reunião aceita a limitação de ordering por order_id enquanto houver um worker, sem garantia global | TRANSCRICAO | [09:12] Diego; [09:13] Diego; [09:13] Larissa |
| ADR-005-ALT | docs/adrs/ADR-005-worker-separado-com-polling.md | Alternativa / trade-off | Worker dentro da API: rejeitado pelo acoplamento a seus reinícios | TRANSCRICAO | [09:09] Bruno; [09:09] Diego; [09:11] Diego |
| ADR-005-CONS | docs/adrs/ADR-005-worker-separado-com-polling.md | Consequência derivada | Positivas: processo de entrega pode ter ciclo de vida separado da API; reutiliza a stack e o banco | TRANSCRICAO | [09:09] Diego; [09:10] Larissa; [09:12] Diego; [09:42] Diego |
| ADR-005-COD | docs/adrs/ADR-005-worker-separado-com-polling.md | Evidência do código | Pontos existentes e limites de integração | CODIGO | src/server.ts; src/config/database.ts; package.json |
| ADR-006-CTX | docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md | Contexto | O OMS já organiza domínios em controllers, services, repositories, routes e schemas | TRANSCRICAO | [09:27] Bruno; [09:28] Bruno; [09:29] Bruno |
| ADR-006-DEC | docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md | Decisão | Organizar webhooks como módulo seguindo esses padrões; reutilizar AppError, Pino, schemas Zod e middleware de erros | TRANSCRICAO | [09:29] Larissa; [09:30] Larissa; [09:36] Larissa |
| ADR-006-DET | docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md | Decisão / restrição | A configuração de webhook usa autenticação normal para qualquer role nesta fase | TRANSCRICAO | [09:32] Bruno; [09:32] Larissa; [09:37] Sofia |
| ADR-006-ALT | docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md | Alternativa / trade-off | Introduzir outro logger: rejeitado por Bruno ao afirmar que Pino já está no projeto e não será adicionado outro | TRANSCRICAO | [09:27] Bruno; [09:29] Bruno; [09:30] Larissa |
| ADR-006-CONS | docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md | Consequência derivada | Positivas: convenções e envelope de erros reconhecíveis; menor diversidade de infraestrutura | TRANSCRICAO | [09:28] Bruno; [09:29] Bruno; [09:30] Larissa |
| ADR-006-COD | docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md | Evidência do código | Integração existente; NotFoundError e ValidationError têm códigos genéricos fixos | CODIGO | src/shared/errors/app-error.ts; src/middlewares/validate.middleware.ts; src/middlewares/error.middleware.ts; src/middlewares/auth.middleware.ts; src/app.ts; src/shared/errors/http-errors.ts |
| RFC-PROP-01 | docs/RFC.md | Proposta baseada em decisão | Outbox MySQL, worker separado, retry/DLQ, HMAC, at-least-once e reuso | TRANSCRICAO | [09:48] Larissa; [09:49] Diego; [09:49] Bruno; [09:49] Sofia |
| RFC-CTX-01 | docs/RFC.md | Contexto de produto | Clientes B2B consultam pedidos repetidamente; necessidade abaixo de 10s | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| RFC-COD-01 | docs/RFC.md | Evidência do código | API modular e transação de status existentes, sem infraestrutura de webhooks | CODIGO | src/app.ts; src/routes/index.ts; src/modules/orders/order.service.ts; prisma/schema.prisma |
| RFC-PROP-02 | docs/RFC.md | Decisão arquitetural | Atomicidade, filtro na inserção e snapshot | TRANSCRICAO | [09:06] Diego; [09:34] Bruno; [09:40] Bruno; [09:52] Larissa |
| RFC-PROP-03 | docs/RFC.md | Decisão arquitetural | Worker único inicialmente, polling 2s, Prisma próprio por processo | TRANSCRICAO | [09:10] Larissa; [09:11] Diego; [09:12] Diego; [09:30] Bruno |
| RFC-PROP-04 | docs/RFC.md | Requisito funcional / segurança | Gestão autenticada e replay administrativo auditado | TRANSCRICAO | [09:21] Sofia; [09:31] Marcos; [09:33] Bruno; [09:34] Marcos; [09:36] Sofia; [09:37] Sofia |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | HTTP síncrono acoplaria transação à disponibilidade remota | TRANSCRICAO | [09:04] Bruno; [09:04] Larissa; [09:06] Diego |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Redis Streams exigiria infraestrutura adicional | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger exigiria canal adicional para avisar processo externo | TRANSCRICAO | [09:09] Bruno; [09:09] Diego; [09:10] Larissa |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Exactly-once aumentaria a coordenação entre as partes | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| RFC-Q-01 | docs/RFC.md | Questão em aberto na reunião | Rate limiting de saída será observado e decidido depois | TRANSCRICAO | [09:38] Diego; [09:39] Diego; [09:39] Larissa |
| RFC-Q-02 | docs/RFC.md | Funcionalidade adiada | Arquivamento fora do escopo; 30 dias não é retenção aprovada | TRANSCRICAO | [09:08] Diego |
| RFC-Q-03 | docs/RFC.md | Lacuna identificada | Contagem das tentativas não concilia cinco intervalos | TRANSCRICAO | [09:17] Diego; [09:17] Larissa; [09:48] Larissa |
| RFC-Q-04 | docs/RFC.md | Lacuna identificada | Ordering sob retry não tem mecanismo definido | TRANSCRICAO | [09:12] Diego; [09:13] Larissa; [09:17] Larissa |
| RFC-Q-05 | docs/RFC.md | Lacuna identificada | Protocolo da rotação de secrets não foi detalhado | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| RFC-Q-06 | docs/RFC.md | Funcionalidade fora de escopo | Email e dashboard não integram esta fase | TRANSCRICAO | [09:37] Larissa; [09:40] Larissa |
| RFC-RISK-01 | docs/RFC.md | Risco derivado | Outbox aumenta carga no banco e pode acumular eventos | TRANSCRICAO | [09:07] Bruno; [09:08] Diego; [09:09] Diego |
| RFC-RISK-02 | docs/RFC.md | Risco derivado | Deduplicação depende do consumidor; DLQ exige intervenção | TRANSCRICAO | [09:18] Diego; [09:25] Diego; [09:25] Sofia |
| RFC-RISK-03 | docs/RFC.md | Risco identificado no código | Faltam vínculo usuário-cliente e controle da atribuição pública de ADMIN | CODIGO | prisma/schema.prisma; src/modules/auth/auth.routes.ts; src/modules/auth/auth.schemas.ts; src/modules/users/user.service.ts |
| RFC-REV-01 | docs/RFC.md | Restrição de processo | Revisão de segurança pré-deploy e revisão do design antes de implementar | TRANSCRICAO | [09:46] Sofia; [09:47] Larissa; [09:50] Larissa |
| FDD-CTX-01 | docs/FDD.md | Contexto técnico | Evitar chamadas HTTP dentro da transação do pedido | TRANSCRICAO | [09:04] Bruno; [09:06] Diego; [09:40] Bruno |
| FDD-OBJ-01 | docs/FDD.md | Objetivos técnicos confirmados | Atomicidade, entrega desacoplada, autenticidade e identificação do evento | TRANSCRICAO | [09:06] Diego; [09:22] Sofia; [09:26] Larissa; [09:30] Larissa |
| FDD-SCOPE-01 | docs/FDD.md | Escopo confirmado | Outbound, gestão, filtros, deliveries, rotação e replay | TRANSCRICAO | [09:02] Marcos; [09:21] Sofia; [09:31] Marcos; [09:33] Bruno; [09:34] Marcos; [09:35] Diego |
| FDD-SCOPE-02 | docs/FDD.md | Exclusões e adiamentos | Sem inbound, email, painel, arquivamento ou escala horizontal nesta fase | TRANSCRICAO | [09:02] Marcos; [09:08] Diego; [09:13] Diego; [09:37] Larissa; [09:39] Larissa; [09:40] Larissa |
| FDD-INT-01 | docs/FDD.md | Evidência do código | Estender changeStatus na transação existente | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Evidência do código | Preservar máquina de estados e regras de estoque | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-03 | docs/FDD.md | Evidência do código | Entrada HTTP existente da mudança de status | CODIGO | src/modules/orders/order.routes.ts; src/modules/orders/order.controller.ts |
| FDD-INT-04 | docs/FDD.md | Evidência do código | Composição e prefixo atual da API | CODIGO | src/app.ts; src/routes/index.ts |
| FDD-INT-05 | docs/FDD.md | Evidência do código | Persistência sem modelos de webhook; UUID com exceção da sequência | CODIGO | prisma/schema.prisma; prisma/migrations/20260519182739_init/migration.sql |
| FDD-INT-06 | docs/FDD.md | Evidência do código | Prisma e bootstrap reutilizáveis por outro processo | CODIGO | src/config/database.ts; src/config/env.ts; src/server.ts; package.json |
| FDD-INT-07 | docs/FDD.md | Evidência do código | JWT identifica usuário e requireRole verifica role | CODIGO | src/middlewares/auth.middleware.ts; src/modules/auth/auth.service.ts |
| FDD-INT-08 | docs/FDD.md | Evidência do código | AppError suporta códigos próprios, validação compartilhada é genérica | CODIGO | src/shared/errors/app-error.ts; src/shared/errors/http-errors.ts; src/middlewares/validate.middleware.ts; src/middlewares/error.middleware.ts |
| FDD-INT-09 | docs/FDD.md | Evidência do código | Pino e correlação HTTP existentes; sem tracing distribuído | CODIGO | src/shared/logger/index.ts; src/middlewares/request-logger.middleware.ts |
| FDD-INT-10 | docs/FDD.md | Evidência do código | Convenções de respostas CRUD e paginação | CODIGO | src/modules/products/product.controller.ts; src/shared/http/response.ts; src/modules/orders/order.schemas.ts |
| FDD-INT-11 | docs/FDD.md | Evidência do código | Estrutura de testes de integração e limpeza do banco | CODIGO | tests/orders.test.ts; tests/auth.test.ts; tests/helpers/factories.ts; tests/setup.ts; vitest.config.ts |
| FDD-DATA-01 | docs/FDD.md | Modelo lógico fundamentado | Configuração, outbox, entrega e DLQ; schema físico pendente | TRANSCRICAO | [09:08] Diego; [09:18] Diego; [09:21] Bruno; [09:21] Sofia; [09:34] Marcos; [09:51] Larissa; [09:52] Larissa |
| FDD-FLOW-01 | docs/FDD.md | Fluxo confirmado e integração proposta | Filtrar, capturar snapshot e inserir com o tx de changeStatus | TRANSCRICAO | [09:06] Diego; [09:34] Bruno; [09:40] Bruno; [09:41] Bruno; [09:41] Diego; [09:52] Larissa |
| FDD-FLOW-02 | docs/FDD.md | Fluxo confirmado com detalhes abertos | Polling, seleção de pendentes, assinatura e entrega | TRANSCRICAO | [09:08] Diego; [09:09] Diego; [09:11] Diego; [09:22] Sofia; [09:30] Bruno; [09:42] Diego |
| FDD-FLOW-03 | docs/FDD.md | Fluxo confirmado com ambiguidade | Backoff nominal e esgotamento para DLQ | TRANSCRICAO | [09:15] Diego; [09:17] Diego; [09:17] Larissa; [09:18] Diego; [09:48] Larissa |
| FDD-FLOW-04 | docs/FDD.md | Fluxo confirmado | Replay administrativo autenticado e auditado | TRANSCRICAO | [09:18] Diego; [09:35] Diego; [09:36] Sofia; [09:36] Larissa |
| FDD-CONV-01 | docs/FDD.md | Proposta derivada do código — não aprovada | Classificação por operação: semântica confirmada, contratos propostos e lacunas; convenções do OMS | CODIGO | src/app.ts; src/modules/products/product.routes.ts; src/modules/products/product.controller.ts; src/shared/http/response.ts; src/modules/orders/order.schemas.ts |
| FDD-CONV-02 | docs/FDD.md | Requisito confirmado / extensão proposta não aprovada | JWT e replay ADMIN confirmados; customer body/path na reunião, query apenas proposta | TRANSCRICAO | [09:32] Larissa; [09:36] Larissa; [09:37] Sofia |
| FDD-API-01 | docs/FDD.md | Contrato proposto baseado em requisito | POST cria configuração e devolve secret gerada | TRANSCRICAO | [09:31] Marcos; [09:32] Larissa |
| FDD-API-02 | docs/FDD.md | Contrato proposto baseado em requisito | PATCH edita URL e status de interesse | TRANSCRICAO | [09:33] Bruno; [09:33] Marcos |
| FDD-API-03 | docs/FDD.md | Contrato proposto baseado em requisito | DELETE remove configuração; semântica de pendentes aberta | TRANSCRICAO | [09:33] Bruno |
| FDD-API-04 | docs/FDD.md | Contrato proposto baseado em requisito | GET lista configurações por customer | TRANSCRICAO | [09:33] Bruno |
| FDD-API-05 | docs/FDD.md | Rota citada / representação proposta | Histórico inclui resultado, payload, resposta e duração | TRANSCRICAO | [09:34] Marcos |
| FDD-API-06 | docs/FDD.md | Rota citada / representação proposta | POST de replay exige ADMIN e retorna à outbox | TRANSCRICAO | [09:35] Diego; [09:36] Sofia; [09:36] Larissa |
| FDD-API-07 | docs/FDD.md | Contrato proposto baseado em requisito | API de rotação preserva secret anterior por 24h | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| FDD-PAYLOAD-01 | docs/FDD.md | Contrato de evento aceito / exemplo | JSON enxuto com identidade, estados, customer e total; sem itens | TRANSCRICAO | [09:43] Diego; [09:44] Bruno; [09:52] Larissa |
| FDD-HEADERS-01 | docs/FDD.md | Headers confirmados | Identificadores, HMAC, timestamp de envio e JSON | TRANSCRICAO | [09:44] Diego; [09:44] Sofia; [09:45] Diego |
| FDD-ERR-01 | docs/FDD.md | Padrão confirmado / exemplos não exaustivos | Prefixo WEBHOOK_ confirmado; três nomes citados não fecham status, schemas ou alcance | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| FDD-ERR-02 | docs/FDD.md | Proposta derivada do código — não aprovada | Separar erros HTTP síncronos de falhas assíncronas; mapeamentos e nomes não aprovados | CODIGO | src/shared/errors/app-error.ts; src/shared/errors/http-errors.ts; src/middlewares/validate.middleware.ts; src/middlewares/error.middleware.ts |
| FDD-ERR-03 | docs/FDD.md | Proposta de nomenclatura baseada em condição confirmada | Nomes para excesso de payload e timeout não foram aprovados | TRANSCRICAO | [09:24] Larissa; [09:42] Diego |
| FDD-RES-01 | docs/FDD.md | Restrições confirmadas | Timeout 10s, payload 64KB, retry limitado e DLQ; sem fallback de email | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:24] Larissa; [09:37] Larissa; [09:42] Diego |
| FDD-RES-02 | docs/FDD.md | Garantia e limitação confirmadas | At-least-once e deduplicação não asseguram sucesso eventual irrestrito | TRANSCRICAO | [09:25] Diego; [09:26] Larissa; [09:18] Diego |
| FDD-OBS-01 | docs/FDD.md | Requisito confirmado / proposta de correlação | Pino, autoria do replay e histórico de entrega | TRANSCRICAO | [09:29] Bruno; [09:34] Marcos; [09:36] Sofia |
| FDD-OBS-02 | docs/FDD.md | Proposta de métricas — não aprovada | Medir latência, resultados, retries e DLQ a partir dos fluxos confirmados | TRANSCRICAO | [09:02] Marcos; [09:15] Diego; [09:18] Diego; [09:34] Marcos |
| FDD-DEP-01 | docs/FDD.md | Restrições de compatibilidade | Mesma stack/banco e reuso; nenhuma dependência nova escolhida | TRANSCRICAO | [09:11] Diego; [09:29] Bruno; [09:30] Bruno; [09:30] Larissa |
| FDD-Q-01 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | B: contagem de envios e cinco intervalos sem interpretação escolhida | TRANSCRICAO | [09:17] Diego; [09:17] Larissa; [09:48] Larissa |
| FDD-Q-02 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | B: ordering sob retry e DLQ; sequenciamento não adotado | TRANSCRICAO | [09:12] Diego; [09:13] Larissa; [09:17] Larissa |
| FDD-Q-03 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | B: rotação 24h sem protocolo escolhido ou aprovação de múltiplas assinaturas | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| FDD-Q-04 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | B: resultados, persistência, recuperação e DLQ indefinidos; batch pode aguardar dimensionamento | TRANSCRICAO | [09:08] Diego; [09:09] Diego; [09:15] Diego; [09:18] Diego |
| FDD-Q-05 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | D: medir abaixo de 10s; marcos propostos sem percentil ou carga inventados | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa; [09:42] Diego |
| FDD-Q-06 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | D: alcance do prefixo e integração proposta via AppError/validação local | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa; [09:29] Bruno |
| FDD-Q-07 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | D: contratos, schemas e criação PENDING pendentes; query é extensão proposta | TRANSCRICAO | [09:21] Sofia; [09:31] Marcos; [09:32] Larissa; [09:33] Bruno; [09:34] Marcos |
| FDD-Q-08 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | B: fan-out, identidade e repetição de replay sem semântica escolhida | TRANSCRICAO | [09:18] Diego; [09:25] Diego; [09:33] Bruno; [09:35] Diego |
| FDD-Q-09 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | B: autorização por customer e ADMIN confiável são dependências futuras, sem modelo aprovado | CODIGO | prisma/schema.prisma; src/modules/auth/auth.routes.ts; src/modules/auth/auth.schemas.ts; src/modules/users/user.service.ts |
| FDD-Q-10 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | D/B/A: ciclo de vida, rejeição do payload e arquivamento; bytes UTF-8 apenas proposta | TRANSCRICAO | [09:08] Diego; [09:21] Sofia; [09:24] Larissa; [09:33] Bruno; [09:52] Larissa |
| FDD-Q-11 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | A: proteção temporal adicional aberta; assinatura do corpo permanece | TRANSCRICAO | [09:22] Sofia; [09:44] Diego |
| FDD-Q-12 | docs/FDD.md | Lacuna / classificação de implementabilidade autoral | A/D: tecnologia de tracing pode aguardar; correlação e registros mínimos exigem definição | CODIGO | src/shared/logger/index.ts; src/middlewares/request-logger.middleware.ts; package.json |
| FDD-AC-01 | docs/FDD.md | Validação derivada | Atomicidade, ausência de evento em rollback e snapshot estável | TRANSCRICAO | [09:06] Diego; [09:40] Bruno; [09:52] Larissa |
| FDD-AC-02 | docs/FDD.md | Validação derivada | Filtro por status evita eventos desnecessários | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno |
| FDD-AC-03 | docs/FDD.md | Validação derivada | Falha externa segue timeout, retry e DLQ | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:42] Diego |
| FDD-AC-04 | docs/FDD.md | Validação derivada | HMAC, HTTPS, limite e rotação | TRANSCRICAO | [09:22] Sofia; [09:23] Sofia; [09:24] Larissa |
| FDD-AC-05 | docs/FDD.md | Validação derivada | Identidade para deduplicar tentativas | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| FDD-AC-06 | docs/FDD.md | Validação derivada | Controle e auditoria do replay | TRANSCRICAO | [09:35] Diego; [09:36] Sofia; [09:36] Larissa |
| FDD-AC-07 | docs/FDD.md | Validação derivada | CRUD, histórico e headers seguem contrato revisado | TRANSCRICAO | [09:31] Marcos; [09:33] Bruno; [09:34] Marcos; [09:44] Diego; [09:44] Sofia |
| FDD-AC-08 | docs/FDD.md | Validação derivada | Validar meta de latência sob condições definidas | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| PRD-CTX-01 | docs/PRD.md | Contexto de produto | Notificar clientes B2B sobre mudanças de status | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| PRD-CTX-02 | docs/PRD.md | Problema relatado | Consulta repetitiva causa integração lenta/cara e risco de migração | TRANSCRICAO | [09:00] Marcos |
| PRD-USE-01 | docs/PRD.md | Cenários confirmados | Integração B2B, escolha de status, diagnóstico e replay | TRANSCRICAO | [09:00] Marcos; [09:33] Marcos; [09:34] Marcos; [09:36] Sofia |
| PRD-MET-01 | docs/PRD.md | Objetivo quantitativo confirmado | Atraso de notificação abaixo de 10 segundos | TRANSCRICAO | [09:02] Marcos |
| PRD-MET-02 | docs/PRD.md | Objetivo qualitativo confirmado | Reduzir dependência de consulta periódica | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| PRD-OUT-01 | docs/PRD.md | Funcionalidade adiada | Email de alerta/fallback não integra a fase | TRANSCRICAO | [09:37] Larissa; [09:38] Marcos; [09:48] Larissa |
| PRD-OUT-02 | docs/PRD.md | Funcionalidade fora de escopo | Dashboard é projeto de frontend separado | TRANSCRICAO | [09:40] Larissa |
| PRD-OUT-03 | docs/PRD.md | Funcionalidade adiada | Arquivamento não tem política aprovada | TRANSCRICAO | [09:08] Diego |
| PRD-OUT-04 | docs/PRD.md | Questão em aberto | Rate limiting de saída será observado antes de decisão | TRANSCRICAO | [09:38] Diego; [09:39] Diego; [09:39] Larissa |
| PRD-OUT-05 | docs/PRD.md | Restrição de escopo | Sem inbound e sem escala horizontal nesta fase | TRANSCRICAO | [09:02] Marcos; [09:13] Diego; [09:13] Larissa |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Notificar mudança de status outbound | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Cadastrar endpoint com URL e status de interesse | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Gerar e devolver secret na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Customer informado explicitamente fora do JWT | TRANSCRICAO | [09:32] Bruno; [09:32] Larissa |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Editar configuração | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Remover configuração | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Listar webhooks por customer | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Filtrar status na inserção da outbox | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno; [09:34] Diego |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Consultar entregas com resultado, payload, resposta e duração | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Rotacionar secret com 24h de validade paralela | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | Replay manual ADMIN e auditado | TRANSCRICAO | [09:35] Diego; [09:36] Sofia; [09:36] Larissa |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | CRUD disponível a qualquer role autenticada nesta fase | TRANSCRICAO | [09:36] Marcos; [09:37] Sofia |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Autenticidade por HMAC-SHA256 e secret por endpoint | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | HTTPS obrigatório | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Payload limitado a 64KB, com erro se exceder | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | At-least-once e deduplicação por X-Event-Id | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Falhas recuperáveis por retry limitado e DLQ | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:42] Diego |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | Estado e evento elegível compartilham transação | TRANSCRICAO | [09:06] Diego; [09:40] Bruno |
| PRD-DEC-01 | docs/PRD.md | Decisões e trade-offs | Outbox, worker, retry/DLQ, HMAC, at-least-once e reuso | TRANSCRICAO | [09:48] Larissa |
| PRD-DEP-01 | docs/PRD.md | Dependência e estimativa | Mesmo banco/stack, três sprints e prazo externo a confirmar | TRANSCRICAO | [09:11] Diego; [09:45] Marcos; [09:46] Larissa; [09:47] Larissa; [09:47] Marcos |
| PRD-DEP-02 | docs/PRD.md | Restrição de processo | Revisão de segurança de pelo menos dois dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia; [09:47] Larissa; [09:49] Sofia; [09:50] Larissa |
| PRD-DEP-03 | docs/PRD.md | Dependência de comunicação | Explicar deduplicação e integração por API aos clientes | TRANSCRICAO | [09:26] Marcos; [09:40] Marcos |
| PRD-RISK-01 | docs/PRD.md | Risco derivado / avaliação qualitativa do autor | Falha de tentativa remota: probabilidade média, confiança baixa; não estima esgotamento em DLQ | TRANSCRICAO | [09:16] Diego; [09:18] Diego; [09:42] Diego; [09:17] Larissa |
| PRD-RISK-02 | docs/PRD.md | Risco derivado / avaliação qualitativa do autor | Vazamento de secret: probabilidade média, confiança baixa; sem frequência ou incidente atribuído ao OMS | TRANSCRICAO | [09:21] Sofia; [09:22] Diego; [09:22] Sofia; [09:46] Sofia |
| PRD-RISK-03 | docs/PRD.md | Risco derivado / probabilidade indeterminada | Crescimento da outbox não comprova backlog ou saturação sem carga e capacidade medidas | TRANSCRICAO | [09:07] Bruno; [09:08] Diego; [09:12] Diego |
| PRD-TEST-01 | docs/PRD.md | Evidência do código / estratégia derivada | Reaproveitar testes de autenticação e pedidos sem afirmar execução | CODIGO | tests/orders.test.ts; tests/auth.test.ts; tests/helpers/factories.ts; tests/setup.ts; vitest.config.ts |
| FDD-EX-01 | docs/FDD.md | Proposta sujeita à aprovação | Histórico de timeout: consulta HTTP 200 distinta da falha assíncrona; campos e nullability propostos | TRANSCRICAO | [09:34] Marcos; [09:42] Diego |
| FDD-EX-02 | docs/FDD.md | Exemplo proposto derivado do código | Replay recusado a OPERATOR: 403 FORBIDDEN existente; repetição de replay permanece indefinida | CODIGO | src/middlewares/auth.middleware.ts; src/shared/errors/http-errors.ts; src/middlewares/error.middleware.ts |
| FDD-ERR-04 | docs/FDD.md | Proposta sujeita à aprovação | AppError e adaptação local de Zod para domínio; middleware HTTP não trata worker separado | CODIGO | src/shared/errors/app-error.ts; src/shared/errors/http-errors.ts; src/middlewares/validate.middleware.ts; src/middlewares/error.middleware.ts; src/modules/orders/order.controller.ts; src/app.ts |
