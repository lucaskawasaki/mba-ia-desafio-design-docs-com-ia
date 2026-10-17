# RFC — Sistema de Webhooks de Notificação de Pedidos

| Metadado | Valor |
| --- | --- |
| Autor da redação | Codex, a partir das fontes do repositório e das instruções do responsável pela entrega |
| Data | 2026-10-16; não representa a data da reunião |
| Status | Proposta para revisão; decisões históricas registradas nos ADRs |
| Revisores indicados | Larissa (Tech Lead), Marcos (Product Manager), Bruno (Engenheiro Pleno, time de Pedidos), Diego (Engenheiro Sênior, time de Plataforma), Sofia (Engenheira de Segurança) |
| Aprovação | Revisão humana deste texto ainda não registrada |

## Resumo executivo

**RFC-PROP-01** — Registrar eventos de mudança de status em uma outbox na mesma transação do pedido. Um worker separado realiza entregas HTTP autenticadas por HMAC, com retry limitado e DLQ. O consumidor deduplica pelo identificador do evento. A proposta consolida as decisões aceitas na reunião; os contratos de implementação ainda precisam das resoluções indicadas abaixo.

## Contexto e problema

**RFC-CTX-01** — Atlas Comercial, MaxDistribuição e Nova Cargo pediram notificações porque a consulta periódica de pedidos torna suas integrações lentas e caras. Marcos relata risco de migração da Atlas. A necessidade declarada é atraso abaixo de 10 segundos; não foram definidos percentil, volume ou método de medição.

**RFC-COD-01** — A API atual concentra a mudança de status em OrderService.changeStatus(), com estoque e histórico transacionais. A composição modular oferece pontos de extensão, mas não há entidades de webhook, outbox ou DLQ no schema. O [FDD](FDD.md) localiza os componentes concretos.

## Proposta técnica

**RFC-PROP-02** — O produtor seleciona configurações interessadas no novo status e grava o snapshot do evento dentro da transação existente. Sem assinante interessado, não há inserção. A chamada externa ocorre fora da transação. Isso separa falha de persistência, que causa rollback, de falha de entrega, que segue retry/DLQ.

**RFC-PROP-03** — O worker roda separado, inicialmente como uma única instância, lendo pendentes antigos a cada 2 segundos. Compartilha banco e configuração com a API, com sua própria instância Prisma. A leitura em ordem de criação não resolve por si só a ordem de recebimento quando há retries.

**RFC-PROP-04** — A API permitirá criar, editar, remover e listar configurações, selecionar status, rotacionar secrets e consultar entregas. O CRUD aceita qualquer role autenticada nesta fase. Replay de DLQ exige ADMIN e auditoria. O detalhamento HTTP fica no FDD.

## Alternativas consideradas

| ID | Alternativa discutida | Motivo do descarte e trade-off |
| --- | --- | --- |
| **RFC-ALT-01** | HTTP síncrono | Evita o worker, mas mantém a transação dependente da latência e disponibilidade do cliente |
| **RFC-ALT-02** | Redis Streams | Acrescenta infraestrutura operacional para um time pequeno; decidiu-se usar MySQL |
| **RFC-ALT-03** | Trigger reativa | Evitaria polling, mas a discussão não encontrou canal nativo adequado para o processo externo |
| **RFC-ALT-04** | Exactly-once | Reduziria a responsabilidade de deduplicação do cliente ao custo de maior coordenação e complexidade |

## Questões em aberto e adiamentos

Assuntos adiados na reunião são distinguidos das lacunas encontradas ao transformar as falas em especificação. O [FDD](FDD.md#questões-de-implementação) concentra as perguntas de implementação.

| ID | Questão | Situação e encaminhamento |
| --- | --- | --- |
| **RFC-Q-01** | Rate limiting de saída | Adiado para observação; não definir limite nesta entrega |
| **RFC-Q-02** | Arquivamento e retenção | Fora do escopo; “30 dias ou assim” não fecha política |
| **RFC-Q-03** | Quantidade de envios | Resolver Q-01 antes de implementar agendamento |
| **RFC-Q-04** | Ordenação durante falhas | Resolver Q-02 antes de prometer ordem de recebimento |
| **RFC-Q-05** | Validade paralela de secrets | Resolver Q-03 com Sofia antes de fechar assinatura e rotação |
| **RFC-Q-06** | Email e dashboard | Email adiado; painel é projeto separado de frontend |

## Impacto e riscos

**RFC-RISK-01** — A solução acrescenta escrita transacional e leitura periódica no MySQL. Backlog e crescimento da tabela precisam de validação; batch e índices físicos não estão fechados. A política de arquivamento não pode ser presumida.

**RFC-RISK-02** — Duplicatas são parte do contrato. Eventos na DLQ exigem operação administrativa, sem promessa de sucesso remoto irrestrito. Os ADRs delimitam essas consequências.

**RFC-RISK-03** — O schema não relaciona usuário a customer. O cadastro público aceita role ADMIN e o service a persiste. Autenticação e requireRole não comprovam isolamento entre clientes nem atribuição confiável do privilégio. Q-09 exige revisão desse pressuposto, sem incluir silenciosamente uma nova feature de autorização.

**RFC-REV-01** — A reunião reserva pelo menos dois dias úteis para revisão de segurança e prevê revisão do design por Larissa com Bruno e Diego antes de codar. A estimativa é três sprints, sem calendário civil definido. A revisão deste RFC ainda está pendente.

## Decisões relacionadas

- [ADR-001 — Outbox no MySQL](adrs/ADR-001-outbox-no-mysql.md).
- [ADR-002 — Retry e DLQ](adrs/ADR-002-retry-com-backoff-e-dlq.md).
- [ADR-003 — HMAC e secret por endpoint](adrs/ADR-003-hmac-e-secret-por-endpoint.md).
- [ADR-004 — At-least-once e event_id](adrs/ADR-004-at-least-once-e-event-id.md).
- [ADR-005 — Worker separado](adrs/ADR-005-worker-separado-com-polling.md).
- [ADR-006 — Padrões do OMS](adrs/ADR-006-reuso-dos-padroes-do-oms.md).

[PRD](PRD.md): escopo e valor de produto. [FDD](FDD.md): contratos, fluxos e validação. [TRACKER](TRACKER.md): fontes por item.
