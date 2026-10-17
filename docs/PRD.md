# PRD — Sistema de Webhooks de Notificação de Pedidos

Status: requisitos extraídos e consolidados para revisão. Criação: 2026-10-16. Revisão documental: 2026-10-17. A implementação não faz parte desta entrega. Detalhes de construção estão no [FDD](FDD.md); escolhas arquiteturais no [RFC](RFC.md) e nos [ADRs](adrs/README.md).

## Resumo e contexto da feature

**PRD-CTX-01** — O OMS passará a notificar sistemas de clientes quando o status de seus pedidos mudar. A feature é outbound: os clientes recebem notificações; não enviam eventos ao OMS por esta funcionalidade.

## Problema e motivação

**PRD-CTX-02** — Atlas Comercial, MaxDistribuição e Nova Cargo solicitaram formalmente notificações. Atualmente consultam pedidos repetidamente, com lentidão e custo de integração relatados por Marcos. Atlas sinalizou possível migração para concorrente se a demanda não for atendida. Não há mensuração de economia, adoção ou probabilidade de churn na transcrição.

## Público-alvo e cenários de uso

**PRD-USE-01** — Integradores dos clientes B2B cadastram endpoints e selecionam os status relevantes, como SHIPPED e DELIVERED. Consultam o histórico para investigar entregas. Administradores do OMS reprocessam eventos da DLQ de forma auditável. Esses cenários não implicam dashboard, envio de email ou permissões por customer já implementadas.

## Objetivos e métricas de sucesso

| ID | Objetivo / métrica | Meta e limite da evidência |
| --- | --- | --- |
| **PRD-MET-01** | Atender à necessidade de notificação rápida; medir atraso entre mudança de status e notificação | **Abaixo de 10 segundos**. Marcos informa o limiar aceito pelos clientes. Percentil, marcos exatos, carga e condições de falha precisam de FDD Q-05 |
| **PRD-MET-02** | Permitir integração orientada a notificações | Sem meta numérica de redução de polling, custos, churn ou adoção; não criar percentuais |

A meta de latência é distinta do timeout de uma tentativa HTTP e do intervalo de polling. Não há SLO aprovado de disponibilidade ou taxa de sucesso. O “99% dos casos” citado por Diego ao discutir deduplicação não é meta de produto.

## Escopo

Inclui os requisitos funcionais e não funcionais abaixo. A garantia de integração é suportada por outbox, entrega assíncrona, retry/DLQ e identificação do evento; o PRD não define schemas HTTP.

### Fora de escopo

| ID | Item | Situação |
| --- | --- | --- |
| **PRD-OUT-01** | Email quando o webhook falha | Adiado, condicionado a avaliação futura |
| **PRD-OUT-02** | Dashboard visual | Fora desta entrega |
| **PRD-OUT-03** | Arquivamento de entregues | Fora desta fase; 30 dias é sugestão informal |
| **PRD-OUT-04** | Rate limiting de saída | Observar e decidir depois |
| **PRD-OUT-05** | Inbound; múltiplos workers e ordering global | Inbound excluído; escala adiada; não há garantia global |

## Requisitos funcionais

Todos os itens desta tabela foram confirmados na reunião. Seus detalhes ainda não decididos estão identificados no FDD.

| ID | Requisito | Critério de aceitação de produto |
| --- | --- | --- |
| **PRD-FR-01** | Notificar mudanças de status | Sistema do cliente recebe informação sobre mudança elegível, respeitando suas assinaturas e o tratamento de falhas |
| **PRD-FR-02** | Cadastrar webhook | Cliente informa URL e seleção de status por API autenticada |
| **PRD-FR-03** | Gerar secret no OMS | Criação devolve a secret gerada; não requer que o cliente a forneça |
| **PRD-FR-04** | Associar configuração ao customer explícito | Identificador informado no body ou path; localização exata sujeita a revisão |
| **PRD-FR-05** | Editar webhook | Operação PATCH disponível; efeitos sobre pendentes dependem do desenho |
| **PRD-FR-06** | Remover webhook | Operação DELETE disponível; não presumir cascatas ou exclusão do histórico |
| **PRD-FR-07** | Listar configurações | GET permite consultar os webhooks de um customer |
| **PRD-FR-08** | Respeitar lista de status | Sem configuração interessada, nenhum evento é inserido na outbox |
| **PRD-FR-09** | Consultar histórico de entregas | Exibir sucesso/falha, payload, resposta e tempo de resposta; limite de 100 não é requisito |
| **PRD-FR-10** | Rotacionar secret via API | Secret anterior permanece válida por 24h; protocolo depende da revisão de segurança |
| **PRD-FR-11** | Reprocessar DLQ | Usuário ADMIN autorizado reenfileira manualmente; autoria é registrada |
| **PRD-FR-12** | Autenticar operações de configuração | ADMIN e OPERATOR podem usar CRUD nesta fase; isso não resolve autorização por customer |

## Requisitos não funcionais

| ID | Requisito confirmado | Validação e limite |
| --- | --- | --- |
| **PRD-NFR-01** | Autenticidade e integridade | Verificação HMAC e adulteração do corpo; detalhes criptográficos no FDD |
| **PRD-NFR-02** | Transporte HTTPS | Rejeitar cadastro de URL HTTP |
| **PRD-NFR-03** | Payload de até 64KB | Erro ao exceder; não truncar. Unidade e momento de verificação pendentes |
| **PRD-NFR-04** | Entrega pode repetir evento | Consumidor usa X-Event-Id; não há exactly-once nem sucesso remoto ilimitado |
| **PRD-NFR-05** | Resiliência | Timeout e retry/DLQ conforme ADR-002/FDD; contagem ainda precisa de resolução |
| **PRD-NFR-06** | Consistência transacional | Nenhuma mudança elegível confirmada sem registro correspondente por falha parcial de inserção |

A meta de desempenho é PRD-MET-01. Não acrescentar throughput, disponibilidade, tempo de retenção ou percentual de sucesso sem fonte.

## Decisões e trade-offs principais

**PRD-DEC-01** — O [RFC](RFC.md) consolida as seis escolhas. A outbox mantém atomicidade com carga adicional no banco; o worker desacopla HTTP com espera e limite de capacidade; retry finito leva à intervenção em DLQ; HMAC exige gestão de secrets; at-least-once requer deduplicação; reuso exige compatibilidade com os componentes atuais. Os fundamentos estão nos ADRs, sem repetir seus detalhes aqui.

## Dependências

**PRD-DEP-01** — A solução depende do MySQL e stack atuais. Larissa estimou três sprints incluindo segurança. Atlas deseja fim de novembro; Marcos ainda confirmaria o prazo. Ano, duração do sprint e calendário não foram informados. Isso é estimativa, não data civil aprovada.

**PRD-DEP-02** — Reservar pelo menos dois dias úteis de Sofia para revisar HMAC e geração de secrets antes do deploy. Larissa também encaminhou revisão do design com Bruno e Diego antes de implementar.

**PRD-DEP-03** — Marcos se dispôs a documentar deduplicação e integração via API no portal. Não há evidência de que esse conteúdo já tenha sido publicado.

## Riscos e mitigação

As probabilidades abaixo são **avaliações qualitativas preliminares do autor, registradas em 2026-10-17**, conforme autorização documental do responsável pela entrega. Não foram atribuídas pelos participantes nem mensuradas na reunião. Não equivalem a percentuais, frequência observada ou previsão para um período determinado.

“Média” é uma categoria provisória de planejamento: o cenário tem mecanismo concreto de ocorrência e antecedente relatado, mas a evidência não permite estimar frequência. A confiança é baixa. Impacto e probabilidade são avaliados separadamente; impacto elevado não implica probabilidade elevada. “Indeterminada” é usada quando faltam dados para uma classificação qualitativa defensável.

| ID | Risco | Probabilidade preliminar e justificativa autoral | Impacto fundamentado | Mitigação e informação para reavaliar |
| --- | --- | --- | --- | --- |
| **PRD-RISK-01** | Cliente indisponível ou lento em uma tentativa de entrega | **Média; confiança baixa.** Há relato de manutenção de duas horas em [09:16] Diego e dependência de sistemas externos. A categoria é julgamento do autor, não estatística histórica | Atraso e novas tentativas; eventual esgotamento pode levar à DLQ, mas sua probabilidade não foi estimada | Decididos: timeout, retry limitado e DLQ ([09:17] Larissa, [09:18] Diego, [09:42] Diego). Resolver Q-01. Reavaliar com frequência/duração das falhas; não transferir a classificação automaticamente para chegada à DLQ |
| **PRD-RISK-02** | Secret comprometida | **Média; confiança baixa.** Vazamento em log de cliente foi relatado em [09:22] Diego. A existência do mecanismo justifica atenção conservadora, sem provar recorrência ou vazamento no OMS | Possibilidade de falsificação de mensagens para o endpoint afetado | Decididos: secret por endpoint, rotação e revisão de segurança ([09:21] Sofia, [09:22] Sofia, [09:46] Sofia). Fechar Q-03. Reavaliar após desenho de geração, armazenamento, distribuição e logs |
| **PRD-RISK-03** | Backlog de entregas e possível degradação com crescimento da outbox | **Indeterminada.** [09:07] Bruno levanta o risco; [09:08] Diego discute índices e batches. Não há volume ou medição de capacidade | Atraso de entrega e maior carga no MySQL se a capacidade não acompanhar a demanda. Crescimento de linhas sem arquivamento não comprova, sozinho, backlog | Índices e batches foram discutidos; validação de capacidade é proposta do autor. Medir taxas de chegada/processamento e tempos de entrega. Arquivamento permanece adiado; sem prazo de retenção inventado |

O TRACKER aponta a base factual dessas avaliações; seus timestamps não atribuem a classificação “média” aos participantes. A aceitação documental do campo probabilidade não representa validação operacional das estimativas.

O risco de autorização observado no código está em RFC-RISK-03/FDD Q-09. Não foi uma decisão da reunião e não autoriza alteração de escopo implícita. A atribuição pública de ADMIN deve ser documentada como dependência de segurança para a implementação futura; nenhum código é corrigido nesta entrega.

## Critérios de aceitação

A aceitação deve verificar cada linha PRD-FR e PRD-NFR e a meta PRD-MET-01. A definição executável depende dos contratos revisados no FDD.

- CRUD e filtro: PRD-FR-02 a PRD-FR-08 e PRD-FR-12.
- Entrega, histórico e recuperação: PRD-FR-01, PRD-FR-09, PRD-FR-11, PRD-NFR-04 a PRD-NFR-06.
- Segurança e rotação: PRD-FR-10, PRD-NFR-01 a PRD-NFR-03.
- Desempenho: PRD-MET-01, após definição do protocolo de medição.
- Exclusões: PRD-OUT-01 a PRD-OUT-05 não podem aparecer como requisitos desta fase.

A presença de um documento não demonstra que a feature já foi validada. Q-01, Q-02, Q-03, Q-04, Q-07, Q-08 e Q-09 do FDD impedem declarar a especificação integralmente pronta para implementação.

## Estratégia de testes e validação

**PRD-TEST-01** — Usar a estrutura de integração existente como referência para cenários futuros de CRUD, autenticação, transação e regressão de estoque. A suíte atual contém oito casos de pedidos e sete de autenticação, usa banco real e apaga dados no setup. Não foi executada neste trabalho documental.

FDD-AC-01 a FDD-AC-08 detalham os cenários derivados dos requisitos: rollback, filtro, snapshot, falha remota, retry/DLQ, assinatura, rotação, duplicatas, replay e latência. Respostas esperadas para casos abertos só devem ser fixadas após revisão. O aceite final inclui a revisão de segurança de PRD-DEP-02.
