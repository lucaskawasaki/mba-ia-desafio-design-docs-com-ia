# ADR-005 — Worker separado com polling de dois segundos

## Status

Aceito na reunião quanto à decisão central; detalhamento pendente onde indicado. Registro documental em 2026-10-16, elaborado com auxílio de IA e sujeito à revisão humana. A transcrição não informa a data civil da reunião. Não há implementação desta feature neste pacote.

## Contexto

**ADR-005-CTX.** A entrega deve ser assíncrona e atender à necessidade de notificações abaixo de 10 segundos. A discussão preferiu usar o banco existente, sem um canal adicional de notificação ao processo.

Fonte: [09:02] Marcos; [09:09] Diego.

## Decisão

**ADR-005-DEC.** Executar worker em processo separado, com polling de 2 segundos dos eventos pendentes mais antigos. Inicialmente haverá um worker. Usar o mesmo banco e DATABASE_URL, com instância Prisma própria por processo.

Fonte: [09:09] Diego; [09:10] Larissa; [09:11] Diego; [09:12] Diego; [09:30] Bruno.

## Detalhes associados

**ADR-005-DET.** A reunião aceita a limitação de ordering por order_id enquanto houver um worker, sem garantia global. Escala horizontal, particionamento por order_id e locks foram adiados. A relação entre essa intenção de ordenação e eventos em retry permanece sem mecanismo definido.

Fonte: [09:12] Diego; [09:13] Diego; [09:13] Larissa.

## Alternativas Consideradas

**ADR-005-ALT.** Worker dentro da API: rejeitado pelo acoplamento a seus reinícios. Trigger como mecanismo reativo: descartada na discussão por não oferecer o canal nativo necessário de notificação externa e exigir improvisação adicional.

Fonte: [09:09] Bruno; [09:09] Diego; [09:11] Diego.

## Consequências

**ADR-005-CONS.** Positivas: processo de entrega pode ter ciclo de vida separado da API; reutiliza a stack e o banco. Negativas: polling acrescenta espera e consultas; um worker pode acumular backlog. O trade-off é operação inicial simples com capacidade e ordenação limitadas. Polling de 2s não demonstra sozinho latência total abaixo de 10s, especialmente com chamadas lentas.

Fonte: [09:09] Diego; [09:10] Larissa; [09:12] Diego; [09:42] Diego.

## Integração existente

**ADR-005-COD.** [src/server.ts](../../src/server.ts) é referência de bootstrap e encerramento; [src/config/database.ts](../../src/config/database.ts) fornece createPrismaClient(); [package.json](../../package.json) ainda não contém script de worker. O novo processo foi discutido, mas não está implementado.

## Questões em aberto

Q-02 cobre ordenação durante falhas; Q-04 cobre seleção, recuperação e batch; Q-05 cobre medição de latência. Tamanho do batch, concorrência interna e tratamento de eventos em processamento não foram fechados.

O [FDD](../FDD.md) concentra o registro de lacunas; o [RFC](../RFC.md) reúne os impactos arquiteturais. A origem de cada item está no [TRACKER](../TRACKER.md).

