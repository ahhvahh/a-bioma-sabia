# Entidade `inbound_update`

**ID:** PST-0102  
**Status:** refinement

## Objetivo

Registrar cada update externo aceito para fornecer idempotência à entrada Telegram.

## Dependências

- [FLW-0001 — Comando Telegram](../../fluxos/comando-telegram.md)
- [PST-0101 — transport_cursor](transport-cursor.md)
- [PST-0103 — request](request.md)

## Estrutura proposta

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `inbound_update_id` | `bigint identity` | não | PostgreSQL | gerado pelo banco | identity |
| `client_id` | `text` | não | configuração | cliente Telegram ativo | copiar |
| `transport` | `text` | não | adaptador | transporte normalizado | `telegram` |
| `transport_update_id` | `bigint` | não | Telegram | `update_id` | inteiro → `bigint` |
| `accepted_at` | `timestamptz` | não | Sabiá | instante do aceite persistente | instante absoluto |
| `request_id` | `uuid` | sim | correlação interna | request criada posteriormente | **proposto para remoção** se a FK ficar somente em `request.inbound_update_id` |

**UNIQUE proposta:** `(client_id, transport, transport_update_id)`.

## Exemplo ilustrativo

| inbound_update_id | client_id | transport | transport_update_id | accepted_at | request_id |
|---:|---|---|---:|---|---|
| 1201 | `bioma` | `telegram` | 48291 | `2026-10-02T23:30:00Z` | `NULL` |

## Origem Telegram

`transport_update_id` é obtido de `Update.update_id`.

## Registro

A tentativa de inserir uma combinação já existente representa update duplicado e não deve gerar nova execução.

## Consumo

É consultada principalmente para:

- verificar idempotência;
- rastrear a origem de uma `request`;
- reconstruir o estado local após restart.

## BLOCKED

Definir a direção física final da correlação com `request`. PST-0002 prefere manter apenas `request.inbound_update_id`.
