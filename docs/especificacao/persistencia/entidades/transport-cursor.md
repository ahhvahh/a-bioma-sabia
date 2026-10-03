# Entidade `transport_cursor`

**ID:** PST-0101  
**Status:** refined

## Objetivo

Persistir o cursor confirmado de leitura de cada transporte/cliente, permitindo que o adaptador retome o consumo sem solicitar novamente updates já aceitos.

## Dependências

- [FLW-0001 — Comando Telegram](../../fluxos/comando-telegram.md)
- [MOD-0002 — Adaptador Telegram](../../modulos/telegram.md)
- [CTR-0010 — Origem e conversão dos dados persistentes](../../contratos/origem-dados-persistencia.md)

## Estrutura proposta

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `client_id` | `text` | não | configuração do cliente | identificador lógico em `telegram.clients` | copiar identificador lógico |
| `transport` | `text` | não | adaptador | transporte normalizado | para Telegram, persistir `telegram` |
| `last_update_id` | `bigint` | não | Telegram `getUpdates` | `update_id` | inteiro do transporte → `bigint` |
| `updated_at` | `timestamptz` | não | Sabiá | relógio no commit | instante absoluto → `timestamptz` |

**PK proposta:** `(client_id, transport)`.

## Exemplo ilustrativo

| client_id | transport | last_update_id | updated_at |
|---|---|---:|---|
| `bioma` | `telegram` | 48291 | `2026-10-02T23:30:00Z` |

## Registro

O cursor só pode avançar na mesma unidade transacional que torna o `inbound_update` correspondente durável.

## Consumo

No startup/polling, o adaptador lê o cursor e calcula:

`offset = last_update_id + 1`

## Restrições

- nunca diminuir `last_update_id`;
- falha antes do commit não altera o cursor;
- um cliente não compartilha cursor com outro.
