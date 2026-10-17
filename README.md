# Sistema de Webhooks de Notificação de Pedidos — Design Docs

## Sobre o desafio

Este repositório documenta o desenho de uma feature de webhooks outbound para o Order Management System existente. A tarefa foi transformar uma reunião técnica e a leitura do código em PRD, RFC, FDD, ADRs e rastreabilidade, mantendo explícita a diferença entre requisito confirmado, decisão, proposta, adiamento e lacuna.

A entrega é documental, em Markdown. O código da aplicação, Prisma, testes, configurações e a transcrição foram preservados. O [enunciado original](docs/ENUNCIADO.md) foi copiado integralmente antes da substituição deste README. A base examinada é o commit e7f6311b96ee3b21bf8cb7077f53a6e4045143eb.

## Situação da entrega

O pacote está redigido para revisão humana. As seis decisões principais estão nos ADRs; propostas de contrato estão identificadas como propostas no FDD. A aprovação da produção dos documentos não foi tratada como aprovação dessas escolhas ainda não decididas.

A [auditoria](docs/AUDITORIA.md) registra verificações e ressalvas. O [registro de questões](docs/FDD.md#questões-de-implementação) concentra o que falta decidir, especialmente contagem de retry, ordering sob falhas, protocolo de rotação, recuperação do worker, contratos e autorização. Dois riscos do PRD têm probabilidade qualitativa preliminar, identificada como avaliação do autor com confiança baixa e justificativa técnica. Não são estatísticas nem decisões da reunião. Não há afirmação de que a feature esteja implementada ou validada em execução.

## Ferramentas de IA utilizadas

- **Codex:** leitura das fontes, análise de consistência, redação e revisão do pacote. A revisão humana dos documentos continua pendente.
- **PowerShell e ripgrep:** inventário e inspeção de código, testes, transcrição e configurações.
- **Git:** identificação da base e verificação de que as alterações ficaram nos documentos.
- **Python:** verificações documentais em memória de IDs, fontes, caminhos, links, seções e exemplos JSON. Não foi criado arquivo de implementação ou de testes para essa auditoria.

Não foram usadas fontes externas para definir requisitos ou decisões. Não foram usados agentes paralelos, outras ferramentas de IA ou modelos adicionais nesta produção.

## Workflow adotado

1. **Contextualização somente leitura:** leitura integral de TRANSCRICAO.md e do README original; inspeção de todos os módulos, persistência, testes e configurações relevantes. A primeira etapa terminou com análise e plano apresentados na conversa, sem arquivos criados.
2. **Autorização de produção:** após “Pode seguir”, preservação do enunciado e uso da matriz de evidências como base.
3. **ADRs:** seis decisões principais em arquivos separados, com consequências e alternativas; ambiguidades mantidas em aberto.
4. **RFC:** síntese arquitetural para revisão, com alternativas descartadas e assuntos adiados.
5. **FDD:** integração com arquivos reais, fluxos, sete exemplos de contrato, matriz de erros, resiliência, observabilidade e critérios técnicos. Propostas de preenchimento foram rotuladas.
6. **PRD:** consolidação do problema, escopo, 12 requisitos funcionais, requisitos não funcionais e meta de atraso abaixo de 10 segundos.
7. **TRACKER:** registro durante a redação e consolidação em tabela única, usando IDs nos documentos e origens verificáveis.
8. **README e auditoria:** registro do processo real, checagens estáticas e correções de consistência.
9. **Revisão dirigida:** análise das ressalvas sem alterar o pacote; posteriormente, sua resposta foi salva em Markdown a pedido do usuário.
10. **Finalização documental autorizada:** após o alinhamento explícito de 2026-10-17, correção de probabilidades qualitativas, justificativa do customer, contratos e erros, exemplos de falha/replay e classificação das lacunas; atualização do TRACKER e nova auditoria.

Os testes da aplicação não foram executados: o setup usa banco real e apaga registros. Os testes descritos no FDD são uma estratégia futura, não resultados obtidos. Não houve commit, push ou publicação do repositório nesta produção.

## Prompts customizados

### 1. Contextualização e extração — trecho do pedido aplicado

O primeiro prompt foi fornecido pelo responsável pela entrega. Trecho relevante, preservando a instrução de separar análise e produção:

~~~text
NESTA PRIMEIRA ETAPA, NÃO CRIE NEM MODIFIQUE DOCUMENTOS.

- Leia integralmente TRANSCRICAO.md e o README.md original.
- Explore src/, prisma/, tests/ e arquivos de configuração somente para leitura.
- Identifique contradições aparentes e como foram resolvidas na reunião.
- Diferencie propostas individuais de decisões efetivamente aceitas.
- Não converta alternativas descartadas em requisitos.
- Não confunda intenção de implementação com comportamento já existente.
- Não invente metas, APIs, payloads, timeouts ou parâmetros.
- Quando o enunciado exigir detalhes não decididos, registre explicitamente
  a lacuna e proponha uma forma de representá-la sem apresentá-la como
  decisão aprovada.
~~~

### 2. Produção e auditoria — adaptação operacional aplicada

O segundo prompt sintetiza as instruções de trabalho aplicadas pelo agente após a autorização. É uma adaptação do pedido e do plano; não é uma citação de uma nova mensagem do usuário.

~~~text
Produza ADRs, RFC, FDD e PRD a partir da matriz de evidências.
Mantenha as lacunas explícitas e os contratos ainda não decididos
identificados como propostas para revisão.

Atribua IDs aos itens e mantenha o TRACKER durante a redação.
Para propostas derivadas, a fonte deve sustentar a origem da proposta,
sem ser apresentada como prova de aprovação.

Confronte os documentos com a transcrição e com os componentes reais:
customer não vem do JWT; estoque não muda em toda transição;
Zod não gera automaticamente códigos WEBHOOK_; requestId não é tracing.

Confira IDs, timestamps e participantes, caminhos, links, exemplos JSON,
seções obrigatórias e preservação dos arquivos da aplicação.
Não preencha probabilidades de riscos ou regras de retry sem evidência.
~~~

### 3. Finalização documental — trecho do alinhamento aplicado

O terceiro prompt foi fornecido pelo responsável e autorizou somente as correções documentais. Trecho literal:

~~~text
Aprovo as correções DOCUMENTAIS obrigatórias identificadas no arquivo `REVISAO-DIRIGIDA-RESSALVAS.md` ou no registro equivalente disponível.

Não aprove automaticamente as soluções técnicas propostas para Q-01, Q-02, Q-03, Q-04, Q-08 ou Q-09.

Esses pontos devem continuar abertos quando não houver decisão conclusiva na transcrição.

Não escolha arbitrariamente parâmetros técnicos, estratégias ou protocolos para aparentar que todas as pendências foram resolvidas.
~~~

## Iterações e ajustes

Foram **cinco ciclos principais**: contextualização e plano; redação integrada; auditoria e correções iniciais; revisão dirigida das ressalvas; finalização documental autorizada e nova auditoria. O usuário direcionou o trabalho e autorizou as correções documentais; as verificações foram feitas pelo agente. Isso não equivale a cinco aprovações humanas nem à aprovação dos protocolos ainda abertos.

| Ciclo | Problema encontrado | Ajuste concreto |
| --- | --- | --- |
| 1 — confronto reunião/código | A fala inicial sugeria customer no JWT; a transcrição corrigiu isso e o código só inclui usuário/email/role | Registrado customer explícito, sem criar associação usuário–customer inexistente |
| 1 — limites das fontes | “5 tentativas” não fecha a relação com cinco intervalos; probabilidades de riscos não foram estimadas | Contagem e probabilidades mantidas como pendências, sem números arbitrários |
| 2 — contratos e erros | A semântica das operações estava definida, mas não todos os paths, schemas e códigos; middleware retorna erros genéricos | Exemplos propostos separados de requisitos; matriz distingue domínio e infraestrutura; WEBHOOK_SECRET_REQUIRED não exige secret do cliente |
| 3 — revisão semântica | O primeiro texto do ADR-002 agrupava email entre assuntos “sem decisão”, embora seu adiamento fosse explícito | Texto corrigido para “Email foi explicitamente adiado” |
| 3 — clareza e apresentação | A descrição de resposta perdida no FDD era ambígua; placeholder de JWT sem marcação podia ser interpretado como HTML | Descrito reenviar após perda da resposta de sucesso e formatado o header como código |

| 4 — revisão dirigida | Probabilidade ausente; query atribuída à fala sobre body/path; integração de WEBHOOK_* pouco detalhada | Ressalvas analisadas e resposta registrada em [docs/REVISAO-DIRIGIDA-RESSALVAS.md](docs/REVISAO-DIRIGIDA-RESSALVAS.md), sem implementar as sugestões |
| 5 — riscos e evidências | Era necessário preencher probabilidade sem inventar histórico | Dois riscos receberam avaliação qualitativa autoral, confiança baixa e justificativa; risco de capacidade permanece indeterminado |
| 5 — contratos e erros | Zod e NotFoundError não produzem automaticamente os códigos de domínio; worker não usa middleware HTTP | Query marcada como proposta; integração futura detalhada, canais separados e exemplos de timeout e replay negado acrescentados |
| 5 — implementabilidade e rastreabilidade | As 12 questões não tinham a mesma urgência; novas explicações precisavam de origem | Classificação B/D/A por componente, preservando Q-01/Q-02/Q-03/Q-04/Q-08/Q-09 abertos; TRACKER e auditoria atualizados |

A auditoria também encontrou um falso negativo causado por encoding na passagem do script de verificação ao Python. O comando foi ajustado para UTF-8; isso não exigiu alterar os documentos ou configurações do projeto.

O registro da revisão dirigida é histórico: suas sugestões técnicas continuam propostas. As probabilidades então ausentes e as correções documentais aprovadas estão atualizadas no PRD, FDD e AUDITORIA; o registro histórico não substitui esse estado atual.

## Como navegar a entrega

| Ordem | Arquivo | Uso |
| --- | --- | --- |
| 1 | [docs/PRD.md](docs/PRD.md) | Problema, público, escopo, requisitos e aceitação de produto |
| 2 | [docs/RFC.md](docs/RFC.md) | Síntese arquitetural, alternativas, riscos e revisão |
| 3 | [docs/adrs/README.md](docs/adrs/README.md) | Índice das seis decisões individuais |
| 4 | [docs/FDD.md](docs/FDD.md) | Fluxos, contratos propostos, integração, erros e questões de implementação |
| 5 | [docs/TRACKER.md](docs/TRACKER.md) | Origem de cada item identificado |
| 6 | [docs/AUDITORIA.md](docs/AUDITORIA.md) | Checklist, resultados e pendências |
| Histórico | [docs/REVISAO-DIRIGIDA-RESSALVAS.md](docs/REVISAO-DIRIGIDA-RESSALVAS.md) | Análise anterior das ressalvas; propostas não automaticamente aprovadas |
| Referência | [TRANSCRICAO.md](TRANSCRICAO.md) | Fonte primária da reunião |
| Referência | [docs/ENUNCIADO.md](docs/ENUNCIADO.md) | README original e critérios de aceite |

A ordem de leitura difere da produção: o PRD apresenta o problema antes das decisões técnicas. Consulte o TRACKER junto do documento que estiver revisando.
