# Ingestão local de mídia

![FLW](https://img.shields.io/badge/FLW-FLW--0006-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Receber um arquivo de um produtor local, persistir o conteúdo e prepará-lo para entrega ao cliente correlacionado.

## Dependências

- [MOD-0008 — Ingestão e armazenamento de mídia](../modulos/ingestao-midia.md)
- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)

## Gatilho

Produtor local conecta ao socket de mídia e envia um objeto MessagePack.

## Pré-condições

- socket em execução;
- processo local autorizado a conectar;
- mensagem usa versão suportada;
- `request_id` conhecido.

## Fluxo principal

1. aceitar conexão Unix local;
2. decodificar um objeto MessagePack;
3. validar `version`, `request_id`, `name`, `content_type` e `data`;
4. localizar a requisição e sua correlação de resposta;
5. iniciar transação SQLite;
6. persistir mídia e obter `media_id`;
7. persistir transmissão `pending` para o cliente/destino da requisição;
8. confirmar a transação;
9. responder MessagePack `accepted` com `request_id` e `media_id`;
10. disponibilizar a transmissão para a fila de entrega;
11. encerrar a conexão.

## Fluxos alternativos

### Requisição desconhecida

Responder `unknown_request` e não persistir conteúdo.

### Payload inválido

Responder `invalid_message`.

### Falha de persistência

Executar rollback e responder `persistence_failed`.

### Conteúdo acima do limite

Responder `media_too_large`; o limite permanece pendente de definição.

## Falhas e tratamento

Falha antes do commit nunca produz `accepted`.

Falha depois do commit e antes de o ACK chegar ao produtor pode levar o produtor a repetir o upload. A política de idempotência de uploads repetidos ainda precisa ser definida.

## Resultado

Mídia e transmissão existem de forma persistente e podem sobreviver a restart do Sabiá.

## Critérios de aceite

- ACK só ocorre após commit;
- transmissão é criada junto com a mídia;
- nenhum arquivo temporário é necessário;
- vários uploads podem usar o mesmo `request_id`;
- falha de conexão não corrompe registro parcialmente persistido;
- idempotência de retry do produtor permanece `BLOCKED`.

## Implementação relacionada

Media Ingest, SQLite e Media Delivery Queue.
