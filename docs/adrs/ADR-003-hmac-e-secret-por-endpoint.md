# ADR-003 — HMAC-SHA256 com secret por endpoint

## Status

Aceito na reunião quanto à decisão central; detalhamento pendente onde indicado. Registro documental em 2026-10-16, elaborado com auxílio de IA e sujeito à revisão humana. A transcrição não informa a data civil da reunião. Não há implementação desta feature neste pacote.

## Contexto

**ADR-003-CTX.** O receptor precisa verificar a origem e integridade dos dados enviados para fora da infraestrutura do OMS. A reunião relata um vazamento anterior de secret em log de cliente.

Fonte: [09:19] Sofia; [09:22] Diego.

## Decisão

**ADR-003-DEC.** Assinar o corpo da requisição com HMAC-SHA256, usando secret exclusiva por endpoint. A secret é gerada pelo OMS e devolvida na criação. Permitir rotação via API com validade paralela da secret anterior por 24 horas.

Fonte: [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:31] Marcos.

## Detalhes associados

**ADR-003-DET.** HTTPS é obrigatório. O header X-Signature transporta a assinatura; os demais headers e o limite de payload estão no FDD. Sofia deve revisar HMAC e geração de secret antes do deploy, com pelo menos dois dias úteis reservados.

Fonte: [09:23] Sofia; [09:44] Diego; [09:46] Sofia; [09:47] Larissa.

## Alternativas Consideradas

**ADR-003-ALT.** Secret global: explicitamente rejeitada porque o vazamento afetaria todos os endpoints. Não houve comparação de OAuth, mTLS ou assinatura assimétrica; nenhuma dessas opções é registrada como alternativa discutida.

Fonte: [09:21] Sofia.

## Consequências

**ADR-003-CONS.** Positivas: comprometimento de uma secret não expõe automaticamente todas as configurações; a rotação prevê tempo de migração. Negativas: cada integração precisa guardar e verificar sua secret; o período de transição exige protocolo claro. O trade-off é compartimentação e integridade com responsabilidade de gestão e verificação distribuída entre OMS e consumidor.

Fonte: [09:20] Sofia; [09:21] Sofia; [09:22] Sofia.

## Questões em aberto

Q-03 deve definir qual secret assina durante as 24h, representação da assinatura, geração, armazenamento e distribuição da secret. Q-11 cobre replay attack: X-Timestamp não prova proteção criptográfica do timestamp, pois a decisão assina o corpo. Nenhum algoritmo adicional, formato hexadecimal/base64 ou janela de tolerância foi aprovado.

O [FDD](../FDD.md) concentra o registro de lacunas; o [RFC](../RFC.md) reúne os impactos arquiteturais. A origem de cada item está no [TRACKER](../TRACKER.md).

