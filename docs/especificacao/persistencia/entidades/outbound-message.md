# Entidade `outbound_message`

**ID:** PST-0106  
**Status:** refinement

## Objetivo

Persistir respostas textuais até confirmação de entrega pelo transporte.

## Dependências

- [CTR-0001 — Comando interno](../../contratos/comando-interno.md)
- [MOD-0002 — Adaptador Telegram](../../modulos/telegram.md)
- [FLW-0001 — Comando Telegram](../../fluxos/comando-telegram.md)

## Estrutura e de-para propostos

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `outbound_message_id` | `bigint identity` | não | PostgreSQL | gerado | identity |
| `request_id` | `text` | não | resultado interno | `request_id` CTR-0001 | copiar |
| `client_id` | `text` | não | `request` | `client_id` | copiar |
| `transport` | `text` | não | `request` | `reply_context.transport` | copiar |
| `destination_id` | `text` | não | `request` | `reply_context.destination_id` | copiar |
| `content` | `text` | não | adaptador | texto convertido do resultado | conversão de apresentação ainda parcialmente BLOCKED |
| `status` | `text` | não | Sabiá | lifecycle de entrega | `pending/delivered/failed` |
| `remote_message_id` | `text` | sim | Telegram | `message_id` da resposta de sucesso | normalizar para string |
| `available_at` | `timestamptz` | sim | Telegram/Sabiá | `retry_after` + instante atual | calcular próxima elegibilidade |
| `last_error_code` | `text` | sim | transporte | código de falha | normalizar código |
| `last_error_message` | `text` | sim | transporte | descrição da falha | texto controlado |
| `created_at` | `timestamptz` | não | Sabiá | criação | instante absoluto |
| `updated_at` | `timestamptz` | não | Sabiá | última alteração | instante absoluto |

## Exemplo ilustrativo

| outbound_message_id | request_id | client_id | transport | destination_id | content | status | remote_message_id |
|---:|---|---|---|---|---|---|---|
| 901 | `req-example-001` | `bioma` | `telegram` | `-100123` | `Serviço operacional` | `delivered` | `452` |

## Registro

Antes do envio, criar a linha em `pending`. Após sucesso remoto, persistir `remote_message_id` e `delivered` na mesma transação.

## Consumo

Entregador busca mensagens `pending` elegíveis por `available_at`. No restart, pendências retornam à etapa de envio.

## BLOCKED

Definir se `content: text` é suficiente para todos os resultados textuais ou se será necessário envelope estruturado persistente.
