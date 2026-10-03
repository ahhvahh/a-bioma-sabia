# Entidade `media_transmission`

**ID:** PST-0110  
**Status:** refinement

## Objetivo

Persistir a fila e o estado de entrega externa de cada mídia.

## Dependências

- [CTR-0007 — Transmissão persistente de mídia](../../contratos/transmissao-midia.md)
- [PST-0108 — media](media.md)
- [PST-0103 — request](request.md)
- [PST-0003 — Claim concorrente de filas persistentes](../claim-concorrente-filas.md)

## Estrutura e de-para

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `transmission_id` | `bigint identity` | não | PostgreSQL | gerado | identity |
| `media_id` | `bigint` | não | `media` | `media_id` | copiar |
| `request_id` | `uuid` | não | `media/request` | `request_id` | copiar e validar UUID v4/correlação |
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
| 701 | 501 | `550e8400-e29b-41d4-a716-446655440000` | `bioma` | `telegram` | `-100123` | `delivered` | 145200 | `453` |

## Registro

Na ingestão local CTR-0008, mídia e transmissão são persistidas antes do ACK ao produtor.

## Consumo

Novas tentativas em `pending` são obtidas pelo claim atômico de PST-0003, realizando `pending → transmitting` antes do streaming. Registros `transmitting` encontrados no recovery seguem CTR-0007. A mídia é carregada por `media_id`.

## BLOCKED

Decidir se a consistência `media.request_id == media_transmission.request_id` será garantida por restrição de banco ou apenas pela aplicação.
