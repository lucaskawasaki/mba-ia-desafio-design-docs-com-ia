# Auditoria documental

Revisão final: 2026-10-17. Produção inicial: 2026-10-16. Base: commit e7f6311b96ee3b21bf8cb7077f53a6e4045143eb. Critérios: [README original preservado](ENUNCIADO.md). Escopo: documentação local; não execução, implantação ou publicação da feature.

## Resultado

As correções documentais autorizadas foram concluídas. O pacote contém seis ADRs, PRD, RFC, FDD, TRACKER e README do processo. Os critérios locais verificáveis foram atendidos no inventário auditado, incluindo dois riscos com probabilidade qualitativa preliminar autoral. **Conformidade documental não significa aprovação técnica, implementação ou prontidão irrestrita para desenvolvimento.**

As 12 questões do FDD continuam abertas, com classificação de impacto por componente. Q-01/Q-02/Q-03/Q-04/Q-08/Q-09 não receberam solução aprovada. Contratos e detalhamentos novos estão separados das decisões históricas. Permanecem limites de verificação sobre o formato específico do curso, a paginação física do RFC e a entrega externa em fork público.

A revisão dirigida foi preservada como registro histórico em [REVISAO-DIRIGIDA-RESSALVAS.md](REVISAO-DIRIGIDA-RESSALVAS.md). Suas sugestões não constituem aprovação; o estado atual está nos documentos revisados e nesta auditoria.

## Verificações estáticas

As verificações foram realizadas por comandos somente de leitura e um script Python executado em memória; não foram criados scripts ou testes no repositório.

| Verificação | Resultado |
| --- | --- |
| Itens identificados nos documentos e no TRACKER | 151 definições, 151 IDs únicos e 151 linhas correspondentes |
| Cobertura das unidades identificadas | 151/151 = 100%; acima do mínimo de 80% |
| Fonte TRANSCRICAO | 128/151 = 84,77%; acima do mínimo de 70% |
| Fonte CODIGO | 23 linhas; acima do mínimo de cinco |
| Timestamps e participantes do TRACKER | Todas as citações correspondem a uma fala com timestamp e nome exatos na transcrição |
| Caminhos de fonte CODIGO | Todos existem |
| Links Markdown locais | Destinos e âncoras verificados |
| Exemplos JSON | 13 blocos analisados, todos com sintaxe válida |
| ADRs | Seis arquivos no padrão obrigatório, com as cinco seções mínimas |
| Código, configurações, testes e transcrição | 59 arquivos protegidos verificados: hashes SHA-256 iguais ao início desta revisão e Git sem diferenças contra o commit base |
| Enunciado preservado | Conteúdo igual ao README da base, desconsiderando apenas normalização CRLF/LF na comparação |
| Git diff --check | Sem erros de whitespace no diff; HEAD permanece no commit base |
| Escopo desta revisão | Seis arquivos Markdown alterados, nenhum arquivo novo; demais 68 arquivos do inventário de 74 permanecem byte a byte iguais ao início desta revisão |
| Testes da aplicação | Não executados; não necessários à entrega documental e usam limpeza de banco no setup |
| RFC | 842 palavras na versão auditada; concisão revisada. Markdown não fixa paginação física, portanto não foi medida quantidade de páginas |

A contagem considera unidades com IDs, incluindo seus exemplos e tabelas subordinadas. Menções repetidas não geram novas unidades. A revisão semântica confrontou também o texto sem IDs próprios com as unidades de origem; a contagem automática sozinha não prova que uma afirmação é verdadeira.

## Checklist contra o enunciado

“Atendido” significa presença e conformidade documental, não implementação ou aprovação humana.

| Critério original | Situação | Evidência / ressalva |
| --- | --- | --- |
| PRD em Markdown no caminho exigido | Atendido | [PRD.md](PRD.md) |
| Todas as seções mínimas do PRD | Atendido | Resumo, problema, público, objetivos, escopo, RF/RNF, trade-offs, dependências, riscos, aceite e testes |
| Pelo menos oito requisitos funcionais da reunião | Atendido | PRD-FR-01 a PRD-FR-12 |
| Objetivo com métrica e meta quantitativa | Atendido com medição pendente | PRD-MET-01: abaixo de 10s; FDD Q-05 ainda precisa do protocolo de medição |
| Pelo menos dois itens explicitamente fora de escopo | Atendido | PRD-OUT-01 a PRD-OUT-05 |
| Pelo menos dois riscos com probabilidade, impacto e mitigação | Atendido documentalmente | PRD-RISK-01/02: média, confiança baixa, avaliação autoral preliminar justificada, sem percentuais ou histórico inventado; PRD-RISK-03 indeterminado |
| RFC em Markdown e seções mínimas | Atendido | Metadados, resumo, contexto, proposta, alternativas, abertos, riscos e ADRs |
| Revisores são participantes da reunião | Atendido | Cinco participantes indicados; não há afirmação de revisão concluída |
| RFC conciso, de duas a quatro páginas | Verificação limitada | Texto conciso de 842 palavras; paginação depende de apresentação, não definida no Markdown |
| Duas alternativas reais descartadas, com trade-offs | Atendido | RFC-ALT-01 a RFC-ALT-04 |
| Dois pontos adiados ou não decididos | Atendido | Rate limiting e arquivamento, além das lacunas de implementação |
| Links para pelo menos dois ADRs | Atendido | Seis links na seção de decisões relacionadas |
| FDD em Markdown e todas as seções mínimas | Atendido | Inclui a seção obrigatória de integração |
| Quatro endpoints com request/response e status codes | Atendido como proposta revisável | Sete operações; quatro possuem exemplos JSON de request/response. GET não tem body e DELETE/204 não inventa payload |
| Matriz de erros WEBHOOK_* | Atendido como proposta revisável | Prefixo confirmado, três exemplos da reunião e dois nomes autorais distinguidos; FDD-ERR-04 explica AppError, Zod e limites do middleware; alcance final em Q-06 |
| Integração com quatro caminhos reais | Atendido | Onze unidades de integração, abrangendo mais de quatro arquivos verificados |
| Observabilidade inclui métricas, logs e tracing | Atendido com pendências explícitas | Logs existentes, métricas propostas e tracing sem implementação/escolha aprovada |
| Entre cinco e oito ADRs com nome obrigatório | Atendido | Seis arquivos ADR-NNN-titulo-em-kebab-case.md |
| Seções Status, Contexto, Decisão, Alternativas, Consequências | Atendido | Verificadas nos seis ADRs |
| Pelo menos cinco das seis decisões principais | Atendido | As seis decisões cobertas |
| Pelo menos um ADR referencia código real | Atendido | ADR-001, ADR-005 e ADR-006 |
| TRACKER no formato de seis colunas | Atendido | ID, Documento, Tipo, Conteúdo (resumo), Fonte, Localização |
| Cobertura mínima de 80% dos itens | Atendido no inventário auditado | 151 unidades identificadas e rastreadas; revisão semântica complementar |
| Pelo menos 70% das linhas com transcrição válida | Atendido | 84,77% |
| Pelo menos cinco linhas de código real | Atendido | 23 |
| README com todas as seções obrigatórias | Atendido | Sobre, ferramentas, workflow, prompts, iterações e navegação |
| Pelo menos uma ferramenta de IA usada | Atendido | Codex; ferramentas auxiliares também discriminadas |
| Dois prompts customizados em blocos de código | Atendido | Três prompts: pedido inicial, adaptação operacional e trecho literal do alinhamento autorizado, com origem declarada |
| Dois ajustes concretos de produção | Atendido | Cinco ciclos registrados; correções concretas de email, customer na query, probabilidade, contratos, erros e lacunas |
| Nenhum requisito/decisão contradiz as fontes | Revisado, com limites explicitados | Propostas, dúvidas e alternativas não foram promovidas a decisões históricas; ver revisão semântica |
| Nenhum arquivo de código citado é inexistente | Atendido | Caminhos e links verificados; nomes lógicos futuros não são apresentados como arquivos existentes |
| Formato específico apresentado no curso | Não verificável | Material específico não disponível; usadas as seções mínimas do enunciado e MADR adaptado |
| Entrega em fork público no GitHub | Não verificado nesta etapa | Produção local; não houve commit, push ou publicação |

## Revisão semântica e correções

- Customer no JWT: a correção de [09:32] Larissa prevalece sobre a proposta inicial de [09:31] Marcos. Body/path são opções da reunião; query da listagem é extensão autoral, fundamentada no schema existente e ainda sujeita à aprovação.
- Estoque: a fala simplificada da reunião foi confrontada com as transições reais; não se afirma débito em toda mudança.
- Prisma: mesma DATABASE_URL não significa a mesma instância em processos diferentes.
- UUID: a decisão da outbox é mantida; OrderNumberSequence é exceção ao comentário “tudo é uuid”.
- Retry: a literalidade de cinco tentativas e cinco intervalos foi preservada, sem escolher uma interpretação não aprovada.
- Ordering: single-worker e seleção por criação não foram apresentados como prova de ordenação sob retry.
- Segurança: secret gerada pelo OMS não foi convertida em campo obrigatório enviado pelo cliente; rotação e timestamp permanecem com lacunas.
- Erros: NotFoundError mantém NOT_FOUND; validate/Zod mantém VALIDATION_ERROR. FDD-ERR-04 propõe adaptação de domínio via AppError, preservando o envelope existente; não afirma conversão automática. O worker separado não passa pela cadeia HTTP Express.
- Email: o primeiro texto do ADR-002 dizia que não havia decisão sobre email; foi corrigido para seu adiamento explícito.
- Contratos: paths, nomes, envelopes e status propostos estão identificados; exemplos não são APIs já existentes.
- Retenção, métricas e prazos: “30 dias ou assim”, “99% dos casos” e fim de novembro não viraram política, SLO ou data civil sem fonte.
- At-least-once: duplicação possível e deduplicação pelo consumidor convivem com retry limitado e recuperação manual; não se promete sucesso remoto irrestrito.

## Correções desta revisão e suas evidências

| Arquivo / unidade | Correção documental | Base verificável | Classificação |
| --- | --- | --- | --- |
| [PRD.md](PRD.md), PRD-RISK-01/02/03 | Dois riscos receberam probabilidade qualitativa, justificativa e confiança baixa; capacidade permanece indeterminada | [09:16] Diego; [09:22] Diego; [09:07] Bruno; [09:08] Diego | Obrigatório corrigir: atendido; avaliações são do autor |
| [FDD.md](FDD.md), FDD-CONV-01/02 | Partes confirmadas, propostas e pendentes dos sete contratos; query não atribuída a body/path | [09:32] Larissa; src/modules/orders/order.schemas.ts; fontes FDD-API-01 a 07 | Obrigatório corrigir: atendido; localização definitiva continua em Q-07 |
| FDD-ERR-01/02/03/04 | Prefixo e exemplos distinguidos de catálogo aprovado; separação síncrono/assíncrono; integração futura com erros reais | [09:28] Bruno; [09:29] Larissa; src/shared/errors/app-error.ts; src/shared/errors/http-errors.ts; src/middlewares/validate.middleware.ts; src/middlewares/error.middleware.ts | Obrigatório corrigir: atendido; alcance final do prefixo continua em Q-06 |
| FDD-Q-01 a 12 | Lacuna, impacto, prioridade, encaminhamento e revisão sugerida por questão | Fontes individuais no [TRACKER](TRACKER.md); código e falas não fornecem soluções finais | Obrigatório corrigir a descrição: atendido; decisões permanecem abertas |
| FDD-EX-01/02 | Histórico de timeout e replay negado a OPERATOR, sem inventar política para replay repetido | [09:34] Marcos; [09:42] Diego; src/middlewares/auth.middleware.ts; src/shared/errors/http-errors.ts; src/middlewares/error.middleware.ts | Recomendado melhorar: realizado como exemplo proposto |
| [ADR-006](adrs/ADR-006-reuso-dos-padroes-do-oms.md) | Complemento da integração real e limites de erros; decisão histórica preservada | Classes de erro e middlewares citados no ADR; FDD-ERR-04 | Recomendado melhorar: realizado; não altera a decisão de reuso |
| [TRACKER.md](TRACKER.md) | Linhas afetadas atualizadas e três novas unidades rastreadas | FDD-EX-01/02, FDD-ERR-04 e evidências locais correspondentes | Obrigatório corrigir: atendido |
| [README.md](../README.md) e esta auditoria | Processo real, terceiro prompt, cinco ciclos, checklist e estatísticas atualizadas | Instruções do responsável, alterações realizadas e verificações locais desta revisão | Obrigatório corrigir: atendido; são fatos do processo, fora do denominador da feature |

Somente estes seis arquivos foram modificados nesta revisão: docs/PRD.md, docs/FDD.md, docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md, docs/TRACKER.md, README.md e docs/AUDITORIA.md. RFC, demais ADRs, índice de ADRs, enunciado e revisão dirigida foram preservados.

## Pendências legítimas e revisão humana

| Grupo | Pendência mantida | Próximo passo e limite |
| --- | --- | --- |
| Bloqueantes do componente afetado | Q-01 contagem; Q-02 ordering; Q-03 rotação; Q-04 processamento/recuperação; Q-08 fan-out e replay; Q-09 autorização | Decisão explícita dos responsáveis antes de implementar os componentes afetados. Nenhuma solução foi aprovada automaticamente |
| Definições antes do desenvolvimento correspondente | Q-05 medição; Q-06 erros; Q-07 contratos; Q-10 ciclo de vida/limite de payload; registros mínimos de Q-12 | Revisar propostas identificadas. O ponto de rejeição do payload e seu efeito transacional em Q-10 são bloqueantes da respectiva regra |
| Decisões que podem permanecer abertas | Arquivamento adiado em Q-10; proteção temporal adicional em Q-11; tecnologia de tracing em Q-12; dimensionamento numérico de batch em Q-04 | Não prometer retenção, proteção completa contra replay, tracing implementado ou capacidade não definida |
| Avaliações autorais | Probabilidade qualitativa dos riscos | Revisar ao obter dados de falhas, gestão de secrets e capacidade. O requisito documental está preenchido; não é estimativa operacional validada |
| Aprovação do pacote | Revisão humana dos contratos, propostas e documentos | A autorização das correções não equivale à aprovação dessas escolhas. Revisão de segurança pré-deploy é atividade futura sobre implementação |
| Limites da auditoria | Template específico do curso; paginação física de 2–4 páginas; fork e publicação pública | Material/renderização/entrega externa não verificados. RFC permanece conciso, com 842 palavras; não declarar contagem física de páginas |

A auditoria não encontrou referências inválidas, JSON inválido, divergência entre IDs e TRACKER ou alterações em arquivos protegidos nas verificações executadas. A revisão semântica manteve decisões, exemplos, propostas e pendências separados. Não foram executados testes da aplicação, operações de banco, commit, push ou publicação.
