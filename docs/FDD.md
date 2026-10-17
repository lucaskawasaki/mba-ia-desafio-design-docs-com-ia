# FDD — Sistema de Webhooks de Notificação de Pedidos

Status: especificação documental para revisão, com pendências explícitas. Criação: 2026-10-16. Revisão documental: 2026-10-17. A feature ainda não existe no código. Decisões históricas: [ADRs](adrs/README.md). Motivação arquitetural: [RFC](RFC.md). Rastreabilidade: [TRACKER](TRACKER.md).

## Contexto e motivação técnica

**FDD-CTX-01** — A mudança de status já coordena persistência, histórico e efeitos de estoque. O webhook precisa registrar a intenção de entrega atomicamente sem manter a transação aberta durante uma chamada a terceiros.

Este documento distingue **decisão/requisito confirmado**, **alternativa descartada**, **fora de escopo/adiado**, **questão em aberto** e **proposta técnica do autor**. Evidências do código são qualificadas como **existente**. Todo detalhamento autoral ainda não decidido recebe o rótulo **Proposta sujeita à aprovação**; sua fonte sustenta a motivação, não a aprovação.

A autorização de 2026-10-17 permite corrigir a documentação. Não aprova as soluções técnicas de Q-01, Q-02, Q-03, Q-04, Q-08 ou Q-09. As decisões históricas dos ADRs foram preservadas; a classificação de criticidade abaixo é avaliação documental do autor.

## Objetivos técnicos

**FDD-OBJ-01** — Preservar atomicidade da mudança elegível e da outbox; desacoplar HTTP; permitir recuperação de falhas; autenticar o evento; identificar duplicatas; reutilizar os padrões do OMS. Critérios observáveis de validação estão ao final.

## Escopo e exclusões

**FDD-SCOPE-01** — Inclui configuração de endpoints, seleção de status, outbox, worker, assinatura, rotação de secret, tentativas, histórico de entregas e replay administrativo.

**FDD-SCOPE-02** — Exclui webhooks recebidos pelo OMS, email de fallback/alerta e painel visual. Arquivamento e escala horizontal foram adiados; rate limiting de saída fica para observação e decisão posterior. Não definir retenção de 30 dias com base na sugestão informal.

## Integração com o sistema existente

Os caminhos abaixo existem no repositório. A integração descrita é futura, sem alteração destes arquivos nesta entrega.

| ID | Arquivo / símbolo | Comportamento existente e integração futura |
| --- | --- | --- |
| **FDD-INT-01** | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts), OrderService.changeStatus() | Prisma.$transaction valida transição, aplica estoque, atualiza pedido e histórico. Inserção da outbox deve compartilhar esse tx antes do commit |
| **FDD-INT-02** | [src/modules/orders/order.status.ts](../src/modules/orders/order.status.ts) | canTransition controla estados. Débito só em PENDING → PAID; reposição ao cancelar PAID/PROCESSING. O webhook não amplia essas transições |
| **FDD-INT-03** | [order.routes.ts](../src/modules/orders/order.routes.ts), [order.controller.ts](../src/modules/orders/order.controller.ts) | PATCH de status é autenticado e validado; controller passa req.user.id ao service. A publicação pertence ao fluxo transacional, não ao controller |
| **FDD-INT-04** | [src/app.ts](../src/app.ts), [src/routes/index.ts](../src/routes/index.ts) | buildControllers e buildApiRouter compõem domínios; base /api/v1. Registrar futuramente o domínio seguindo esse padrão |
| **FDD-INT-05** | [prisma/schema.prisma](../prisma/schema.prisma), [migration.sql](../prisma/migrations/20260519182739_init/migration.sql) | MySQL, relações, IDs UUID das entidades e sequência inteira. Novos modelos/migrações serão trabalho de implementação; não existem aqui |
| **FDD-INT-06** | [database.ts](../src/config/database.ts), [env.ts](../src/config/env.ts), [src/server.ts](../src/server.ts), [package.json](../package.json) | createPrismaClient, configuração validada e bootstrap são referências; worker separado precisa de ciclo de vida próprio. Não há script de worker |
| **FDD-INT-07** | [auth.middleware.ts](../src/middlewares/auth.middleware.ts), [auth.service.ts](../src/modules/auth/auth.service.ts) | JWT contém sub/email/role; authenticate e requireRole existem. Não extrair customer do token |
| **FDD-INT-08** | [AppError](../src/shared/errors/app-error.ts), [classes HTTP](../src/shared/errors/http-errors.ts), [validate](../src/middlewares/validate.middleware.ts), [errorMiddleware](../src/middlewares/error.middleware.ts) | Erros de domínio podem herdar AppError. Zod convertido por validate gera VALIDATION_ERROR; prefixo WEBHOOK_ não surge automaticamente |
| **FDD-INT-09** | [logger](../src/shared/logger/index.ts), [requestLogger](../src/middlewares/request-logger.middleware.ts) | Pino registra requestId, usuário e duração HTTP. Não há tracing distribuído nem correlação do worker implementados. Secret não consta das regras explícitas de redação |
| **FDD-INT-10** | [product.controller.ts](../src/modules/products/product.controller.ts), [response.ts](../src/shared/http/response.ts), [order.schemas.ts](../src/modules/orders/order.schemas.ts) | Referências para 201/200/204 e envelope paginado. A adoção nos contratos abaixo é proposta, não decisão da reunião |
| **FDD-INT-11** | [orders.test.ts](../tests/orders.test.ts), [auth.test.ts](../tests/auth.test.ts), [factories.ts](../tests/helpers/factories.ts), [setup.ts](../tests/setup.ts), [vitest.config.ts](../vitest.config.ts) | Supertest/Vitest usam Prisma real, setup apaga dados. Referência para testes futuros; não executados na produção documental |

A aplicação atual é modular, não uma implementação de event bus. A existência de transação não comprova todas as garantias de concorrência possíveis; este desenho não altera as regras atuais de estoque.

## Persistência e fluxos detalhados

### Modelo lógico

**FDD-DATA-01** — Os conceitos abaixo têm origem na reunião; não constituem um schema Prisma aprovado.

| Conceito | Informação sustentada pela reunião | Ainda não definido |
| --- | --- | --- |
| Configuração | URL, secret por endpoint, customer, estado ativo, status de interesse | Constraints, defaults, relação usuário–customer, estrutura da rotação |
| Outbox | UUID, snapshot JSON, status de processamento, ordem por criação | Uma linha por evento ou por destino, schema, índices físicos, agendamento e recuperação |
| Entrega | Sucesso/falha, payload, resposta e tempo de resposta | Modelo de tentativa versus entrega, truncamento de resposta, paginação e retenção |
| DLQ separada | Payload, motivo de falha, timestamp; replay para pendente | Relações, transação de transferência, destino do registro após replay |

Os nomes de tabelas webhook_outbox e webhook_dead_letter foram discutidos. Nenhuma tabela desse domínio existe no schema atual. “Batch pequeno” não determina tamanho numérico.

### Criação transacional do evento

**FDD-FLOW-01**

1. O fluxo existente autentica, valida e chama changeStatus com pedido, novo status e usuário.
2. Dentro do tx, localizar o pedido e executar as verificações e efeitos atuais de estoque.
3. Atualizar status e histórico; selecionar as configurações do customer interessadas no status de destino. A semântica do estado ativo e de mudanças concorrentes de configuração precisa de Q-10.
4. Sem interessados, concluir a transação sem inserir evento.
5. Havendo interessados, gerar identidade UUID e snapshot do evento, persistindo a intenção de entrega com o mesmo tx. A distribuição entre destinos depende de Q-08.
6. Falha de persistência da outbox provoca rollback de todo o tx. Só após commit o worker poderá entregar.

A função publishWebhookEvent(tx, order, fromStatus, toStatus) é a interface proposta por Bruno e apoiada por Diego; não existe no código. “Função pura” foi a expressão da reunião, mas uma função que escreve via tx tem efeito de persistência. O requisito verificável é receber o cliente transacional existente, sem publicar fora dele.

### Processamento pelo worker

**FDD-FLOW-02**

1. Inicializar processo próprio e Prisma conectado ao mesmo banco.
2. A cada 2 segundos, buscar pendentes antigos em batch pequeno.
3. Preparar o corpo a partir do snapshot e a assinatura HMAC; não reconstruir o pedido a partir de seu estado atual.
4. Executar chamada HTTP com os headers definidos e timeout de 10 segundos.
5. Em sucesso, marcar entrega; em falha, encaminhar para política de retry/DLQ. Registrar informações de histórico.

Seleção de eventos já elegíveis para retry, critério de sucesso HTTP, marcação “processando” e retomada após queda são Q-04. O roteiro não aprova um algoritmo de claim, lock ou concorrência interna.

### Retry, DLQ e replay

**FDD-FLOW-03** — Falha exige nova tentativa conforme política limitada. A sequência nominal é 1m, 5m, 30m, 2h e 12h. A reunião diz “5 tentativas”; Q-01 impede montar uma tabela definitiva tentativa → horário. Não adicionar jitter, teto diferente ou retry indefinido. Ao esgotar a política resolvida, persistir o evento na DLQ separada, com motivo e timestamp.

**FDD-FLOW-04** — O replay autentica usuário, exige ADMIN, identifica o registro e o recoloca na outbox como pendente, registrando quem executou. Reset de contagem, preservação de event_id, replay concorrente/repetido e destino da linha da DLQ são Q-08; não presumir sucesso desses casos.

## Contratos públicos

### Convenções e grau de aprovação

**FDD-CONV-01** — **Proposta sujeita à aprovação.** Os exemplos tornam o desenho concreto, mas nenhum contrato completo de webhook está implementado ou integralmente aprovado. Paths de CRUD/rotação, nomes camelCase, envelopes, paginação e status de sucesso são derivados do estilo atual. Datas, UUIDs, URLs e valores são dados fictícios de exemplo.

| Operação / item | Parte confirmada na reunião | Parte proposta, sujeita à aprovação | Pendências que afetam o desenvolvimento |
| --- | --- | --- | --- |
| Criação — FDD-API-01 | POST, URL, seleção de status e devolução da secret gerada; customer fora do JWT ([09:31] Marcos; [09:32] Larissa) | Path, nomes dos campos, body exato, resposta e 201 | Q-07: obrigatoriedade/defaults; Q-09: acesso ao customer |
| Edição — FDD-API-02 | PATCH ([09:33] Bruno) | Path, campos editáveis, resposta e 200 | Q-07/Q-10: patch vazio, active, customer e pendentes; Q-09: autorização |
| Exclusão — FDD-API-03 | DELETE ([09:33] Bruno) | Path e 204 sem corpo | Q-10: exclusão física/lógica, histórico e pendentes; Q-09: autorização |
| Listagem — FDD-API-04 | GET por customer ([09:33] Bruno) | Filtro em query, paginação, representação e 200 | Q-07: localização definitiva do customer; Q-09: visibilidade |
| Deliveries — FDD-API-05 | GET /webhooks/:id/deliveries e informações de histórico ([09:34] Marcos) | Prefixo da API, schema de sucesso/falha, paginação e 200 | Q-07: schema; Q-09: acesso; Q-10: retenção |
| Replay — FDD-API-06 | POST /admin/webhooks/dead-letter/:id/replay, ADMIN e auditoria ([09:35] Diego; [09:36] Sofia; [09:36] Larissa) | Body, resposta de reenfileiramento e 200 | Q-08: identidade, contagem e concorrência; Q-09: atribuição confiável de ADMIN |
| Rotação — FDD-API-07 | API de rotação com coexistência de 24h ([09:21] Sofia; [09:22] Sofia) | Verbo, path, schema e 200 | Q-03: protocolo; Q-07: contrato; Q-09: acesso |

A tabela classifica cada parte do contrato, sem transformar nomes de campos ou códigos HTTP propostos em decisões da reunião. Sua rastreabilidade é a dos itens FDD-API-01 a FDD-API-07 e dos padrões existentes em FDD-INT-04/FDD-INT-10.

**FDD-CONV-02** — Usar `Authorization: Bearer <JWT>` nos exemplos e JSON nos requests com corpo. A decisão de [09:32] Larissa é informar customer no body ou path, sem extraí-lo do JWT. **Proposta sujeita à aprovação:** customerId no body da criação segue uma dessas opções. Já customerId na query da listagem é uma terceira opção, derivada da listagem existente em [order.schemas.ts](../src/modules/orders/order.schemas.ts), e não uma decisão dessa fala.

O exemplo de listagem permanece proposto para revisão: sua adoção exige confirmar essa extensão ou substituí-la por identificação no path. Nenhuma opção foi aprovada automaticamente nesta correção documental. Autorização de deliveries e rotação permanece em Q-09; erros seguem a matriz adiante.

### Criar configuração

**FDD-API-01**

**Proposta sujeita à aprovação:** POST /api/v1/webhooks → 201 Created. Não aceitar secret do cliente como requisito; ela é gerada pelo OMS. Request:

~~~json
{
  "customerId": "11111111-1111-4111-8111-111111111111",
  "url": "https://cliente.example/notificacoes",
  "statuses": ["SHIPPED", "DELIVERED"]
}
~~~

Response ilustrativa; valor e representação da secret não são especificação criptográfica:

~~~json
{
  "id": "22222222-2222-4222-8222-222222222222",
  "customerId": "11111111-1111-4111-8111-111111111111",
  "url": "https://cliente.example/notificacoes",
  "statuses": ["SHIPPED", "DELIVERED"],
  "secret": "<secret-gerada-pelo-OMS>"
}
~~~

Default do estado ativo, unicidade da URL e quantidade de endpoints não foram definidos. Não inferir “secret exibida uma única vez” como requisito aprovado.

### Editar configuração

**FDD-API-02**

**Proposta sujeita à aprovação:** PATCH /api/v1/webhooks/22222222-2222-4222-8222-222222222222 → 200 OK.

~~~json
{
  "url": "https://cliente.example/notificacoes-v2",
  "statuses": ["DELIVERED"]
}
~~~

Response:

~~~json
{
  "id": "22222222-2222-4222-8222-222222222222",
  "customerId": "11111111-1111-4111-8111-111111111111",
  "url": "https://cliente.example/notificacoes-v2",
  "statuses": ["DELIVERED"]
}
~~~

Semântica de patch vazio, edição de customer, estado ativo e efeito sobre eventos pendentes: Q-07/Q-10.

### Remover configuração

**FDD-API-03**

**Proposta sujeita à aprovação:** DELETE /api/v1/webhooks/22222222-2222-4222-8222-222222222222. Request sem body. Response 204 No Content, sem body, seguindo o controller de produtos existente. Não há payload JSON a inventar para uma resposta 204. Exclusão física/lógica, cascatas e impacto sobre eventos/histórico continuam em Q-10.

### Listar por customer

**FDD-API-04**

**Proposta sujeita à aprovação:** GET /api/v1/webhooks?customerId=11111111-1111-4111-8111-111111111111&page=1&pageSize=20. Request sem body. Response 200 OK:

~~~json
{
  "data": [{
    "id": "22222222-2222-4222-8222-222222222222",
    "customerId": "11111111-1111-4111-8111-111111111111",
    "url": "https://cliente.example/notificacoes",
    "statuses": ["SHIPPED", "DELIVERED"]
  }],
  "pagination": {"page": 1, "pageSize": 20, "total": 1, "totalPages": 1}
}
~~~

Envelope e parâmetros são **Proposta sujeita à aprovação** baseada em paginated() e schemas atuais, não limites aprovados para webhooks. O filtro em query é uma extensão proposta, conforme FDD-CONV-02. Lista vazia, acesso ao customer e visibilidade de secret precisam de fechamento em Q-07/Q-09.

### Consultar histórico de entregas

**FDD-API-05**

Rota relativa citada: GET /webhooks/:id/deliveries. **Proposta sujeita à aprovação — montagem:** GET /api/v1/webhooks/22222222-2222-4222-8222-222222222222/deliveries. Request sem body. Response proposta 200 OK:

~~~json
{
  "data": [{
    "success": true,
    "payload": {
      "event_id": "33333333-3333-4333-8333-333333333333",
      "event_type": "order.status_changed",
      "timestamp": "2026-01-01T12:00:00.000Z",
      "order_id": "44444444-4444-4444-8444-444444444444",
      "order_number": "ORD-000001",
      "from_status": "PROCESSING",
      "to_status": "SHIPPED",
      "customer_id": "11111111-1111-4111-8111-111111111111",
      "total_cents": 10000
    },
    "response": {"statusCode": 200, "body": "ok"},
    "responseTimeMs": 120
  }],
  "pagination": {"page": 1, "pageSize": 20, "total": 1, "totalPages": 1}
}
~~~

Nomes, representação de response e paginação são propostos. Status 200 acima é exemplo de resposta do receptor, não definição de quais respostas contam como sucesso. “Últimos 100” foi exemplo de Marcos, não tamanho de página ou retenção aprovado.

### Exemplo de histórico com timeout

**FDD-EX-01** — **Proposta sujeita à aprovação.** [09:34] Marcos exige histórico de sucesso/falha e [09:42] Diego define timeout de 10s. O exemplo abaixo representa consulta bem-sucedida ao histórico de uma tentativa que falhou; HTTP 200 é o status proposto do GET de deliveries, não um status recebido do destinatário.

Request: GET /api/v1/webhooks/22222222-2222-4222-8222-222222222222/deliveries, com JWT, sem body. Response ilustrativa 200:

~~~json
{
  "data": [{
    "success": false,
    "payload": {
      "event_id": "33333333-3333-4333-8333-333333333333",
      "event_type": "order.status_changed",
      "timestamp": "2026-01-01T12:00:00.000Z",
      "order_id": "44444444-4444-4444-8444-444444444444",
      "order_number": "ORD-000001",
      "from_status": "PROCESSING",
      "to_status": "SHIPPED",
      "customer_id": "11111111-1111-4111-8111-111111111111",
      "total_cents": 10000
    },
    "response": null,
    "responseTimeMs": null,
    "error": {
      "code": "WEBHOOK_DELIVERY_TIMEOUT",
      "message": "Delivery request timed out"
    }
  }],
  "pagination": {"page": 1, "pageSize": 20, "total": 1, "totalPages": 1}
}
~~~

O cenário ilustra ausência de resposta HTTP. `null` não significa duração zero: não houve resposta cujo tempo pudesse ser registrado nesse campo. Nullabilidade, campo error e representação final dependem de Q-07; armazenamento e evolução da tentativa dependem de Q-04. O nome WEBHOOK_DELIVERY_TIMEOUT é autoral. O exemplo não fixa contador, data de retry nem critério de esgotamento.

### Replay administrativo

**FDD-API-06**

Rota relativa confirmada: POST /admin/webhooks/dead-letter/:id/replay. **Proposta sujeita à aprovação — montagem:** POST /api/v1/admin/webhooks/dead-letter/55555555-5555-4555-8555-555555555555/replay, com JWT ADMIN. Request proposto:

~~~json
{}
~~~

Response proposta 200 OK:

~~~json
{
  "deadLetterId": "55555555-5555-4555-8555-555555555555",
  "status": "pending"
}
~~~

A resposta informa reenfileiramento, não recebimento pelo cliente. Forma da resposta e literal pending são **Proposta sujeita à aprovação**. O 200 pressupõe que o reenfileiramento já terminou; não declara sucesso remoto. Repetição/conflito do replay, reset de tentativas e identidade resultante dependem de Q-08. Não se define nesta entrega retorno 200/409 nem idempotência para replay repetido.

**FDD-EX-02** — Exemplo de recusa derivado do código existente, a integrar futuramente na rota de replay. Request: mesmo POST de replay, body {}, com JWT válido de role OPERATOR. `requireRole('ADMIN')` gera ForbiddenError; o middleware de erros o representa como HTTP 403:

~~~json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Insufficient permissions"
  }
}
~~~

Origem: [auth.middleware.ts](../src/middlewares/auth.middleware.ts), [http-errors.ts](../src/shared/errors/http-errors.ts) e [error.middleware.ts](../src/middlewares/error.middleware.ts). O formato já existe na infraestrutura; a rota de replay ainda não existe. A recusa acontece antes do reenfileiramento. Isso não resolve a dependência de atribuição confiável de ADMIN em Q-09 nem renomeia o erro compartilhado para WEBHOOK_*.

### Rotacionar secret

**FDD-API-07**

Método/path não decididos. **Proposta sujeita à aprovação:** POST /api/v1/webhooks/22222222-2222-4222-8222-222222222222/rotate-secret. Request:

~~~json
{}
~~~

Response proposta 200 OK:

~~~json
{
  "secret": "<nova-secret-gerada-pelo-OMS>",
  "previousSecretValidUntil": "2026-01-02T12:00:00.000Z"
}
~~~

O exemplo supõe rotação às 12:00 UTC do dia anterior, apenas para ilustrar 24h. Não fixa formato da secret. O endpoint não está pronto para implementação até resolver Q-03, inclusive qual chave assina durante a transição.

### Evento outbound e headers

**FDD-PAYLOAD-01** — O objeto payload apresentado no exemplo de deliveries representa o corpo JSON outbound discutido. timestamp é ISO 8601 do evento; snapshot é produzido na inserção. event_type usa o exemplo order.status_changed. Não incluir items. Detalhes do pedido podem ser consultados na rota existente GET /api/v1/orders/:id. Esse exemplo não estabelece versão de schema, campos adicionais ou regras para criação do pedido.

**FDD-HEADERS-01**

| Header | Semântica confirmada | Não definido |
| --- | --- | --- |
| X-Event-Id | UUID do evento criado na outbox | Identidade no replay/fan-out |
| X-Signature | HMAC-SHA256 do corpo | Codificação e protocolo de rotação |
| X-Timestamp | Timestamp do envio | Formato exato e proteção contra alteração |
| X-Webhook-Id | ID do endpoint/configuração | Modelo físico |
| Content-Type | application/json | Nenhum parâmetro adicional aprovado |

O verbo HTTP outbound não foi explicitado. POST é proposta coerente com o envio de JSON, não decisão registrada. O receptor valida a assinatura do corpo; não foi aprovada fórmula que concatene timestamp, IDs ou outros headers ao material assinado.

## Matriz de erros previstos

**FDD-ERR-01** — WEBHOOK_NOT_FOUND, WEBHOOK_INVALID_URL e WEBHOOK_SECRET_REQUIRED foram exemplos de [09:28] Bruno, não um catálogo de contratos aprovado. O prefixo foi fechado por [09:29] Larissa; os cenários e mapeamentos abaixo são **Proposta sujeita à aprovação**. SECRET_REQUIRED não significa que o cliente deve enviar secret no cadastro.

**FDD-ERR-02**

| Código de domínio | Situação | Canal e status | Origem / grau de definição |
| --- | --- | --- | --- |
| WEBHOOK_NOT_FOUND | Configuração procurada não existe | Síncrono; HTTP 404 proposto | Nome citado; 404 segue convenção atual, mas NotFoundError emite NOT_FOUND |
| WEBHOOK_INVALID_URL | URL inválida ou sem HTTPS | Síncrono; HTTP 400 proposto | Nome citado; HTTPS confirmado. Zod com validate() emite VALIDATION_ERROR até haver adaptação específica |
| WEBHOOK_SECRET_REQUIRED | Secret necessária à assinatura ausente | Assíncrono no cenário proposto; sem status HTTP da API de gestão | Nome citado; cenário interno ainda não aprovado. Não exigir secret no request de cadastro |
| WEBHOOK_PAYLOAD_TOO_LARGE | Evento excede 64KB | Canal e status pendentes de Q-10 | Condição confirmada; nome autoral. Ponto de validação e efeito sobre a transação não foram escolhidos |
| WEBHOOK_DELIVERY_TIMEOUT | Chamada excede 10s | Assíncrono; sem status HTTP da API de gestão | Timeout confirmado; nome autoral. Falha da tentativa, seguida da política de retry ainda dependente de Q-01/Q-04 |

**FDD-ERR-03** — Os dois últimos nomes permitem revisar a matriz sem fingir que foram discutidos. Outros códigos, falhas HTTP remotas e conflitos de replay dependem de Q-04/Q-06/Q-08. Não confundir status retornado pelo endpoint receptor com status da API de gestão.

Exemplo proposto de envelope, usando a classe existente e a exigência de HTTPS:

~~~json
{
  "error": {
    "code": "WEBHOOK_INVALID_URL",
    "message": "Webhook URL must use HTTPS",
    "details": [{"path": "url", "message": "HTTPS is required"}]
  }
}
~~~

Erros compartilhados hoje são 400 VALIDATION_ERROR, 401 UNAUTHORIZED, 403 FORBIDDEN, 409 CONFLICT para P2002, 404 NOT_FOUND para P2025 e 500 INTERNAL_SERVER_ERROR para falha não tratada. A proposta é preservá-los como infraestrutura e usar WEBHOOK_* para o domínio, **sujeita a Q-06**. Não afirmar que o middleware existente atende literalmente “prefixo para tudo”.

### Integração futura com a infraestrutura de erros

**FDD-ERR-04** — **Proposta sujeita à aprovação.** A implementação futura pode reutilizar AppError e o middleware HTTP central, com adaptação explícita no domínio:

1. Para recurso inexistente, um erro de webhook baseado em AppError pode fornecer 404 e WEBHOOK_NOT_FOUND. `NotFoundError('Webhook')` existente altera apenas a mensagem e continua com código NOT_FOUND; seu construtor não permite substituir esse código.
2. Para URL inválida/sem HTTPS, uma validação específica do módulo pode preservar os detalhes Zod e convertê-los em erro de domínio 400/WEBHOOK_INVALID_URL antes de delegar ao tratamento central. Usar somente um refinamento Zod com `validate()` atual resulta em VALIDATION_ERROR. BadRequestError aceita código customizado; ValidationError não.
3. Erros de domínio síncronos chegam ao middleware via o padrão controller/next existente; `errorMiddleware` serializa AppError usando statusCode, errorCode, message e details. Não é necessário supor uma troca global do envelope para exemplificar a integração.
4. Erros de autenticação/autorização e erros Prisma genéricos conservam seus códigos atuais no desenho proposto. A fronteira “domínio WEBHOOK_* versus infraestrutura compartilhada” ainda requer Q-06; não é redefinição histórica da fala “prefixo para tudo”.
5. O worker separado não tem o request/response Express nem a cadeia `next(err)`. Falhas de tentativa precisam de tratamento próprio que registre resultado e use a política de retry/DLQ; não podem ser consideradas tratadas automaticamente pelo middleware HTTP. Estrutura de captura, persistência e recuperação continua em Q-04. Sem retorno HTTP ao chamador original de changeStatus após commit.

Base existente: [app-error.ts](../src/shared/errors/app-error.ts), [http-errors.ts](../src/shared/errors/http-errors.ts), [validate.middleware.ts](../src/middlewares/validate.middleware.ts), [error.middleware.ts](../src/middlewares/error.middleware.ts), [order.controller.ts](../src/modules/orders/order.controller.ts) e [app.ts](../src/app.ts). O desenho é documental: não cria classes nem altera qualquer middleware.

## Estratégias de resiliência

**FDD-RES-01** — Timeout HTTP: 10 segundos. Limite de payload: 64KB, com erro, sem truncamento. Retry: sequência nominal e lacuna do ADR-002. Fallback operacional previsto: persistência em DLQ e replay manual; email foi excluído. Não há circuit breaker, jitter, retry infinito ou política de redirects aprovada.

**FDD-RES-02** — O cliente deduplica pelo evento persistido. Uma resposta de sucesso perdida pode levar o worker a reenviar: esse é um cenário de validação derivado da garantia, não implementação existente. O sistema não promete que um endpoint permanentemente indisponível receberá o evento; esgotamento leva à DLQ.

## Observabilidade

**FDD-OBS-01** — Logs devem reutilizar Pino. Registrar quem fez replay é requisito confirmado. **Proposta sujeita à aprovação:** correlacionar event_id, webhook_id e tentativa no worker, como detalhamento derivado desses identificadores e da necessidade de diagnóstico, ainda sem schema de log aprovado. O logger HTTP atual, sozinho, não registra o resultado assíncrono da entrega.

**FDD-OBS-02** — **Proposta sujeita à aprovação — métricas:** atraso entre mudança e notificação; resultados e duração das entregas; tentativas e eventos em DLQ. Servem para verificar necessidade de latência, recuperação e histórico. Nomes, agregação, labels, alertas e metas adicionais não foram definidos. A única meta de produto fundamentada é abaixo de 10s, cuja medição depende de Q-05.

Tracing é um tópico exigido pelo enunciado, não uma decisão da reunião. **Não existe tracing distribuído identificado na aplicação examinada.** requestId em logs HTTP não comprova propagação pelo worker. Estratégia, biblioteca, spans e retenção permanecem em Q-12. Não inserir OpenTelemetry ou outro fornecedor como escolha aprovada. Revisar redação de secrets antes de produzir logs do novo domínio; o reuso do Pino não cobre automaticamente novos campos sensíveis.

## Dependências e compatibilidade

**FDD-DEP-01** — A feature usa a stack existente e o banco atual. O desenho exige persistência adicional, mas não determina biblioteca HTTP, criptografia auxiliar, fila externa ou versões novas. A leitura do código base referencia o commit e7f6311b96ee3b21bf8cb7077f53a6e4045143eb. As convenções camelCase atuais e snake_case do evento proposto devem ser distinguidas nos contratos.

## Questões de implementação

Todas as questões abaixo continuam **em aberto**. A classificação de implementabilidade é avaliação do autor em 2026-10-17; não muda o status das decisões da reunião. **B — Bloqueante para implementação** significa bloqueio do componente afetado por segurança, consistência ou garantia indefinida. **D — Definição necessária antes do desenvolvimento** exige fechar contrato ou detalhamento antes de construir a parte correspondente. **A — Decisão que pode permanecer aberta** admite adiamento explícito sem promessa de comportamento.

“Proposta sujeita à aprovação” não equivale a decisão confirmada. Os interlocutores são sugestões pelos papéis da reunião. As fontes de cada questão constam no TRACKER e sustentam a pergunta, não uma resposta inexistente.

| Questão / ID | Lacuna e evidência de base | Impacto / classificação do autor | Encaminhamento documental e limite da proposta | Revisão sugerida |
| --- | --- | --- | --- | --- |
| Q-01 — **FDD-Q-01** | Cinco tentativas versus cinco intervalos ([09:17] Diego; [09:17] Larissa; [09:48] Larissa) | **B:** impede definir contador, horários e esgotamento | Registrar as leituras “cinco envios totais” e “inicial mais cinco retries” para decisão, sem escolher uma. A primeira exige esclarecer os intervalos; a segunda altera “total cinco”. Nenhuma foi aprovada nesta revisão | Larissa e Diego |
| Q-02 — **FDD-Q-02** | Ordering por pedido no single-worker, sem regra durante retry ([09:12] Diego; [09:13] Larissa; [09:17] Larissa) | **B:** seleção por criação não comprova ordem de recebimento após falhas | Definir se eventos posteriores esperam e o que ocorre ao atingir DLQ; também esclarecer desempate/ordem causal. Sequenciamento por pedido/destino é apenas alternativa da revisão dirigida, sem adoção; não prometer ordem de processamento remoto | Larissa e Diego |
| Q-03 — **FDD-Q-03** | Secret antiga válida em paralelo por 24h, sem protocolo ([09:21] Sofia; [09:22] Sofia) | **B:** migração pode quebrar verificação da assinatura | Fechar qual secret assina, codificação, geração/armazenamento e rotações sucessivas. Assinaturas múltiplas mencionadas na revisão dirigida continuam proposta, não protocolo escolhido | Sofia |
| Q-04 — **FDD-Q-04** | Pendentes/processando, batch, falhas e DLQ sem algoritmo completo ([09:08] Diego; [09:09] Diego; [09:15] Diego; [09:18] Diego) | **B:** recuperação após queda, resultados HTTP e passagem à DLQ podem perder ou prender trabalho. Batch numérico pode permanecer aberto para dimensionamento | Definir resultado de tentativa, seleção elegível, persistência e retomada. Não adotar automaticamente política 2xx, claim/lease, concorrência ou prazo de recuperação. FDD-ERR-04 descreve a fronteira do tratamento, não resolve esse protocolo | Diego e Bruno |
| Q-05 — **FDD-Q-05** | Meta abaixo de 10s sem marcos, carga ou percentil ([09:02] Marcos; [09:10] Larissa; [09:42] Diego) | **D:** antes da instrumentação e teste de desempenho; não bloqueia todo trabalho independente | **Proposta sujeita à aprovação:** medir commit da mudança → recebimento no receptor de teste, separando fila e duração HTTP. Medir commit → resposta é proxy distinto. Carga, percentil e condições de falha continuam abertos, sem novos valores | Marcos, Larissa e Diego |
| Q-06 — **FDD-Q-06** | Prefixo WEBHOOK_ e reuso de componentes que emitem códigos genéricos ([09:28] Bruno; [09:29] Larissa; [09:29] Bruno; FDD-INT-08) | **D:** antes de fechar contrato de erros e validação | **Proposta sujeita à aprovação:** erros de domínio específicos via AppError e adaptação local da validação, preservando erros compartilhados. FDD-ERR-04 detalha a integração; nenhuma alteração de código ou aprovação do alcance do prefixo | Bruno e Larissa |
| Q-07 — **FDD-Q-07** | Paths, schemas, defaults, paginação e evento de criação não fechados ([09:21] Sofia; [09:31] Marcos; [09:32] Larissa; [09:33] Bruno; [09:34] Marcos; FDD-INT-01) | **D:** contratos precisam de semântica definida antes de implementar os respectivos endpoints | Usar a tabela FDD-CONV-01 para revisar cada parte. Query na listagem é extensão proposta. A publicação discutida é em changeStatus; não incluir automaticamente criação PENDING nem declará-la explicitamente descartada | Bruno, Marcos e Larissa |
| Q-08 — **FDD-Q-08** | Evento versus destinos, event_id no replay e reenfileiramentos repetidos/concorrentes ([09:18] Diego; [09:25] Diego; [09:33] Bruno; [09:35] Diego) | **B:** afeta modelo de dados, deduplicação e efeitos do replay | Não escolher granularidade, preservação de event_id, reset de contador, idempotência ou destino da linha original. O exemplo de sucesso pressupõe reenfileiramento concluído; o de 403 precede esse fluxo. Resultado de replay repetido permanece sem status/schema fechado | Diego e Bruno |
| Q-09 — **FDD-Q-09** | Falta vínculo usuário–customer; cadastro público aceita ADMIN; acesso a deliveries/rotação não definido (prisma/schema.prisma; src/modules/auth/auth.routes.ts; src/modules/auth/auth.schemas.ts; src/modules/users/user.service.ts) | **B:** para concluir autorização e segurança da exposição | Documentar dependência de autorização por customer e atribuição confiável de ADMIN. A correção futura do problema não é implementada aqui. Não escolher modelo de vínculo nem tratar CRUD autenticado como prova de isolamento | Sofia e Larissa |
| Q-10 — **FDD-Q-10** | Edição/exclusão/active, limite de 64KB e retenção ([09:08] Diego; [09:21] Sofia; [09:24] Larissa; [09:33] Bruno; [09:52] Larissa) | **D:** ciclo de vida; **B:** ponto de rejeição do payload e seu efeito transacional; **A:** arquivamento adiado | **Proposta sujeita à aprovação:** medir o corpo JSON serializado em bytes UTF-8. Conversão de KB em bytes, verificação antes/depois do commit e consequências não foram escolhidas. Definir efeitos sobre pendentes/histórico sem alterar o snapshot; não fixar retenção | Bruno, Diego e Sofia |
| Q-11 — **FDD-Q-11** | Corpo assinado e timestamp em header ([09:22] Sofia; [09:44] Diego) | **A:** proteção temporal adicional pode permanecer aberta se não for prometida prevenção completa de replay | Documentar limite do timestamp e distinguir deduplicação de frescor autenticado. Alterar material assinado para incluir headers é nova decisão de protocolo, não correção automática; a assinatura do corpo permanece a decisão histórica | Sofia |
| Q-12 — **FDD-Q-12** | Pino e requestId existentes, sem tracing distribuído (src/shared/logger/index.ts; src/middlewares/request-logger.middleware.ts; package.json) | **A:** ferramenta/stack de tracing; **D:** registros mínimos e correlação do worker | **Proposta sujeita à aprovação:** correlação por evento, endpoint e tentativa em logs, com autoria do replay conforme requisito. Não chamar correlação de tracing implementado nem escolher biblioteca/fornecedor. FDD-OBS-01/02 delimitam métricas e logs propostos | Diego e Larissa |

A classificação impede transformar todas as pendências em uma barreira uniforme ao pacote documental. Q-01/Q-02/Q-03/Q-04/Q-08/Q-09 continuam sem solução aprovada; retenção adiada e tecnologia de tracing podem continuar abertas. Para iniciar uma parte do desenvolvimento, revisar suas dependências específicas e os contratos aplicáveis.

## Critérios de aceite técnicos

Critérios derivados das decisões; são especificação de testes futuros, não testes implementados ou executados.

| ID | Dado / quando / resultado esperado |
| --- | --- |
| **FDD-AC-01** | Havendo endpoint interessado, mudança confirmada persiste evento com estado daquela mudança; falha de inserção reverte status/histórico/estoque; alteração posterior não muda o snapshot |
| **FDD-AC-02** | Sem assinatura para o status, não inserir evento; com assinatura, selecionar os destinos pertinentes, após fechar fan-out |
| **FDD-AC-03** | Timeout de 10s registra falha; validar sequência e DLQ após resolver Q-01/Q-04, incluindo queda durante processamento |
| **FDD-AC-04** | Rejeitar HTTP, validar assinatura e adulteração do corpo, rejeitar payload excessivo; verificar 24h de transição após Q-03/Q-10 |
| **FDD-AC-05** | Tentativas do mesmo evento levam sua identidade persistida; receptor de teste deduplica. Replay exige expectativa própria após Q-08 |
| **FDD-AC-06** | OPERATOR não executa replay; ADMIN autorizado reenfileira e autoria é auditada, considerando a revisão de atribuição de roles em Q-09 |
| **FDD-AC-07** | Exercitar criação/edição/exclusão/listagem, campos do histórico e todos os headers após fechamento de schemas; não inventar body em GET ou 204 |
| **FDD-AC-08** | Medir atraso de notificação e confrontar com abaixo de 10s, usando protocolo de Q-05; não considerar polling prova suficiente |

Regressão das transições e estoque deve reutilizar a cobertura descrita em FDD-INT-11. Testes de assinatura/adulteração, timeout, queda, duplicata e replay são cenários derivados dos requisitos acima. Não executar a suíte em banco não isolado: o setup existente remove dados.

## Riscos e mitigação

| Base rastreável | Risco / impacto | Mitigação decidida ou proposta |
| --- | --- | --- |
| FDD-FLOW-01 / FDD-RES-01 | Falha da outbox impede mudança de status | Rollback é decisão; testar atomicidade e dimensionar escrita é validação proposta |
| FDD-Q-01 / FDD-Q-02 | Implementações divergem sobre tentativas e ordem | Resolver contagem e ordenação antes de codar as respectivas regras |
| FDD-Q-03 / FDD-Q-11 | Assinatura/rotação incompatível ou promessa indevida de proteção | Revisão de Sofia; não publicar protocolo não definido |
| FDD-Q-09 | Papel autenticado não comprova direito sobre customer ou origem confiável do ADMIN | Revisão de autorização como dependência; nenhuma correção de código nesta entrega |
| FDD-OBS-01 / FDD-INT-09 | Novos logs podem incluir secrets sem redação correspondente | Revisar campos e política de logs antes da implementação; proposta derivada do código |

A revisão de segurança pré-deploy e o prazo mínimo reservado estão em PRD-DEP-02. As probabilidades não foram estimadas nas fontes. O PRD agora separa duas avaliações qualitativas preliminares do autor, com confiança baixa, da evidência histórica; nenhuma probabilidade numérica foi criada. Resolver pendências técnicas exige decisão explícita; a autorização das correções documentais não aprova os protocolos em aberto.
