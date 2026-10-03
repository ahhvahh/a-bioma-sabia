# Entidade `media_transmission`

**ID:** PST-0110  
**Status:** refinement

## Objetivo

Persistir a fila e o estado de entrega externa de cada mídia.

## Dependências

- [CTR-0007 — Transmissão persistente de mídia](../../contratos/transmissao-midia.md)
- [PST-0108 — media](media.md)
- [PST-0103 — request](request.md)

## Estrutura e de-para

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `transmission_id` | `bigint identity` | não | PostgreSQL | gerado | identity |
| `media_id` | `bigint` | não | `media` | `media_id` | copiar |
| `request_id` | `text` | não | `media/request` | `request_id` | copiar e validar correlação |
| `client_id` | `text` | não | `request` | `client_id` | copiar |
| `transport` | `text` | não | `request` | `transport` | copiar |
| `destination_id` | `text` | não | `request` | `destination_id` | copiar |
| `status` | `text` | não | Delivery Queue | lifecycle | `pending/transmitting/delivered/failed` |
| `created_at` | `timestamptz` | não | Sabiá | criação | instante absoluto |
| `updated_at` | `timestamptz` | não | Sabiá | alteração | instante absoluto |
| `remote_message_id` | `text` | sim | Telegram | `message_id` de sucesso | normalizar para string |
| `last_error` | `text` | sim | transporte/Sabiá | falha | mensagem/código controlado |
| `streamed_bytes` | `bigint` | não | stream local | bytes consumidos | contador iniciado em 0 por tentativa |

## Exemplo ilustrativo

| transmission_id | media_id | request_id | client_id | transport | destination_id | status | streamed_bytes | remote_message_id |
|---:|---:|---|---|---|---|---|---:|---|
| 701 | 501 | `req-example-001` | `bioma` | `telegram` | `-100123` | `delivered` | 145200 | `453` |

## Registro

Na ingestão local CTR-0008, mídia e transmissão são persistidas antes do ACK ao produtor.

## Consumo

A fila seleciona `pending` e `transmitting` elegíveis. A mídia é carregada por `media_id`.

## BLOCKED

Decidir se a consistência `media.request_id == media_transmission.request_id` será garantida por restrição de banco ou apenas pela aplicação.
