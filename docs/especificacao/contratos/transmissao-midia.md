# Transmissão persistente de mídia

![CTR](https://img.shields.io/badge/CTR-CTR--0007-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Definir a fila persistente de mídia destinada ao cliente, permitindo retomar transmissões após reinício do serviço sem depender de arquivos temporários.

## Dependências

- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0012 — Ingestão persistente de mídia por Unix socket](../../adr/processamento/ingestao-midia-socket-messagepack.md)
- [CTR-0006 — Mídia persistida](midia-persistida.md)
- [CTR-0009 — Ingestão fracionada de mídia por Unix socket](ingestao-midia-fracionada.md)
- [MOD-0002 — Adaptador Telegram](../modulos/telegram.md)

## Tipo

`interface`

## Registro persistente

Cada entrega de mídia possui registro no SQLite com, no mínimo:

- `transmission_id: integer` — identificador persistente da transmissão;
- `media_id: integer` — conteúdo persistido em CTR-0006;
- `request_id: string` — correlação com a requisição;
- `client_id: string` — cliente lógico responsável pela entrega;
- `transport: string` — transporte de destino;
- `destination_id: string` — destino opaco no transporte;
- `status: pending | transmitting | delivered | failed`;
- `created_at: timestamp`;
- `updated_at: timestamp`;
- `remote_message_id: string | null`;
- `last_error: string | null`.

Nome, tipo e BLOB pertencem ao registro de mídia referenciado por `media_id`.

## Estados

### pending

Mídia persistida e aguardando tentativa de transmissão.

### transmitting

Existe tentativa de envio em andamento ou a execução anterior terminou sem persistir conclusão definitiva.

No startup, `transmitting` volta a ser elegível para envio. Isso preserva semântica `at-least-once`: uma confirmação remota perdida localmente pode produzir reenvio.

### delivered

O transporte confirmou o recebimento e a confirmação foi persistida.

### failed

A transmissão não pode continuar automaticamente.

## Criação

A transmissão deve ser criada na mesma transação lógica que torna a mídia recebida disponível para entrega quando o upload vier de CTR-0008.

O produtor só recebe ACK de sucesso depois de mídia e transmissão estarem persistidas.

## Envio

1. selecionar transmissão `pending` ou elegível para retry;
2. carregar metadados e a fonte persistida por `media_id`;
3. quando a mídia for fracionada, verificar que o arquivo lógico está completo e ler os chunks por ordem de `sequence_id`;
4. marcar `transmitting`;
5. fornecer um stream contínuo ao adaptador correspondente;
6. aguardar confirmação remota;
7. persistir `remote_message_id` quando fornecido;
8. persistir `delivered`.

A política de retenção do BLOB depois de `delivered` pertence a CTR-0006.

## Recovery no startup

Antes de considerar a fila recuperada:

1. consultar transmissões `pending` e `transmitting`;
2. verificar a existência do `media_id` correspondente;
3. se a mídia existir, tornar a transmissão elegível para nova tentativa;
4. se a mídia não existir, marcar `failed` com `media_missing`.

Não existe reconciliação com `/tmp` ou filesystem para mídia persistida.

## Erros

- `media_missing`;
- `media_delivery_failed`;
- `media_confirmation_persistence_failed`;
- `transport_not_available`;
- `destination_not_available`.

## Regras e restrições

- `media_id` precisa existir antes da transmissão;
- reinício retoma `pending` e `transmitting`;
- confirmação remota é persistida antes de `delivered`;
- falha ambígua permite reenvio;
- conteúdo binário não é duplicado dentro da tabela de transmissão;
- chunks não são entregues individualmente ao transporte externo;
- mídia fracionada só inicia entrega quando estiver completa;
- binário não aparece em logs.

## Compatibilidade

O registro usa `transport` e `destination_id`, portanto não depende exclusivamente do Telegram.

## Critérios de aceite

- restart retoma transmissões pendentes;
- mídia persistida continua disponível sem filesystem temporário;
- várias mídias da mesma requisição possuem transmissões independentes;
- falha ambígua pode ser reenviada;
- confirmação remota é persistida;
- ausência de `media_id` produz falha explícita.
