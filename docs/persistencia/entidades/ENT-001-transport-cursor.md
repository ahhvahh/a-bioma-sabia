<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-001 — Entidade `sabia.transport_cursor`

**ID canônico:** `ENT-001` · **ID histórico:** `PST-0101`  
**Banco e schema:** `abioma_sabia` / `sabia` (**propostos**)  
**Estado documental de origem:** `refined` · **Estado do DDL:** proposta; não há migração executada verificada.

## Objetivo

Persistir o cursor confirmado de leitura de cada transporte/cliente, permitindo que o adaptador retome o consumo sem solicitar novamente updates já aceitos.

## Dependências

- [FLUX-001 (FLW-0001) — Comando Telegram](../../fluxos/FLUX-001-comando-telegram.md)
- [ARQ-008 (MOD-0002) — Adaptador Telegram](../../arquitetura/ARQ-008-adaptador-telegram.md)
- [CONTR-010 (CTR-0010) — Origem e conversão dos dados persistentes](../../contratos/CONTR-010-origem-dados-persistencia.md)

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

## Verificação de consistência

O identificador lógico `client_id` vem do adaptador configurado, não de uma FK para tabela de bots. A chave `(client_id, transport)` isola os cursores por cliente e transporte; o commit do update e do cursor deve ser atômico. Consulte [entrada recebida](ENT-002-inbound-update.md) e [relacionamentos](../PERS-001-relacionamentos.md).

## Chaves, índices e integridade física

- **PK:** `(client_id, transport)`; não criar índice redundante da PK.
- **Consistência:** `last_update_id` só avança na mesma transação que aceita `inbound_update`.
- **Consulta:** `(client_id, transport)` já é indexado pela PK.

**Estado do DDL:** proposta de tabela; nenhum SQL de migração foi executado.

## Revisão de consistência e pendências

**Consistência transacional:** o avanço do cursor e a aceitação durável de `inbound_update` devem integrar uma única transação; esta documentação não comprova transações executadas.

Todas as decisões físicas pendentes permanecem registradas em [pendências PostgreSQL](../pendencias.md). A ficha descreve uma **especificação documental**, não comprova instalação do banco nem aprovação do DDL.

## Rastreabilidade

Fonte imediatamente anterior: [PST-0101 — `sabia.transport_cursor`](transport-cursor.md). A fonte histórica em `develop`, seus exemplos e os campos originais permanecem rastreáveis no arquivo legado.
