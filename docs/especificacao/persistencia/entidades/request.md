# Entidade `request`

**ID:** PST-0103  
**Status:** refinement

## Objetivo

Persistir a requisição normalizada entregue ao Core, independente do formato específico do transporte.

## Dependências

- [CTR-0001 — Comando interno](../../contratos/comando-interno.md)
- [FLW-0001 — Comando Telegram](../../fluxos/comando-telegram.md)
- [PST-0102 — inbound_update](inbound-update.md)

## Estrutura e de-para propostos

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `request_id` | `text` | não | Adaptador/Sabiá | ID gerado na entrada | normalizar para string; formato ainda não definido |
| `inbound_update_id` | `bigint` | sim | PostgreSQL | PK de `inbound_update` | copiar FK quando a origem for update persistido |
| `client_id` | `text` | não | configuração/adaptador | cliente lógico que recebeu | copiar |
| `principal_id` | `text` | não | Telegram → adaptador | Telegram User ID | normalizar para string |
| `transport` | `text` | não | adaptador | `reply_context.transport` | copiar valor normalizado |
| `destination_id` | `text` | não | Telegram → adaptador | Chat ID | normalizar para string opaca |
| `source_message_id` | `text` | sim | Telegram | `message_id`, quando existir | normalizar para string |
| `command` | `text` | não | adaptador | comando identificado no update | token principal normalizado |
| `arguments` | `text[]` | não | adaptador | argumentos tokenizados | lista ordenada → `text[]`; sem argumentos = array vazio |
| `status` | `text` | não | Sabiá | estado de processamento | domínio ainda BLOCKED |
| `received_at` | `timestamptz` | não | adaptador | instante de aceite | normalizar para instante absoluto |
| `updated_at` | `timestamptz` | não | Sabiá | última alteração | relógio do serviço |

## Exemplo ilustrativo

| request_id | inbound_update_id | client_id | principal_id | transport | destination_id | source_message_id | command | arguments | status | received_at |
|---|---:|---|---|---|---|---|---|---|---|---|
| `req-example-001` | 1201 | `bioma` | `778899` | `telegram` | `-100123` | `451` | `status` | `[]` | `<BLOCKED>` | `2026-10-02T23:30:01Z` |

## Origem Telegram

A documentação atual confirma conceitualmente `update_id`, User ID, Chat ID, `message_id`, comando e argumentos. O caminho bruto exato desses valores dentro do objeto Telegram ainda não está especificado em `/docs`.

## Consumo

A entidade é a fonte de correlação para:

- criação de jobs;
- ingestão de mídia por socket usando `request_id`;
- construção de transmissões e respostas;
- autorização de visibilidade de jobs.

## BLOCKED

- definir estados/transições de `request.status`;
- definir formato/algoritmo de geração de `request_id`;
- registrar em contrato os caminhos exatos dos campos do objeto Telegram usados pelo adaptador.
