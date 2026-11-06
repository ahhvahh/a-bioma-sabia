<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Entidade `sabia.inbound_update`

> **Conteúdo lido e migrado para:** [ENT-002 — documento canônico](ENT-002-inbound-update.md). Esta ficha `PST` permanece como fonte histórica; o DDL segue não verificado.

![Tipo](https://img.shields.io/badge/Tipo-entidade-0550ae?style=flat-square) ![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

**Banco proposto:** `abioma_sabia` · **Schema proposto:** `sabia` · **Tabela:** `sabia.inbound_update`

**ID:** PST-0102  
**Status:** refinement

## Objetivo

Registrar cada update externo aceito para fornecer idempotência à entrada Telegram.

## Dependências

- [FLW-0001 — Comando Telegram](../../especificacao/fluxos/comando-telegram.md)
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
| `raw_update` | `jsonb` | não | Telegram | objeto `Update` integral, sem token | persistir antes do avanço do cursor |
| `processing_status` | `text` | não | Sabiá | `accepted/processing/completed/ignored/failed` | transição controlada |

**UNIQUE proposta:** `(client_id, transport, transport_update_id)`.

## Exemplo ilustrativo

| inbound_update_id | client_id | transport | transport_update_id | accepted_at | raw_update | processing_status |
|---:|---|---|---:|---|---|---|
| 1201 | `bioma` | `telegram` | 48291 | `2026-10-02T23:30:00Z` | `{ "update_id": 48291 }` | `accepted` |

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

**Proposta desta reorganização:** conservar a FK apenas em `request.inbound_update_id`, sem coluna redundante `inbound_update.request_id`. Esta escolha ainda requer aprovação da direção física final.

## Regra de recuperação — decisão de 21/10/2026

A decisão de 21/10/2026 exige o objeto Telegram Update integral em `raw_update` e um ciclo controlado em `processing_status`. O quadro de campos acima incorpora ambos e mantém a unicidade por cliente, transporte e atualização.

Inserir `raw_update` e avançar `transport_cursor.last_update_id` na **mesma transação**. O offset só avança após commit. Reprocessamentos após queda leem `raw_update` localmente. Um claim/lock por update impede processamento concorrente, e resultado existente não deve gerar segunda execução. `raw_update` contém dados pessoais: restringir acesso, excluir dos logs e definir purge seguro depois de todos os trabalhos correlacionados.

## Verificação de consistência

O exemplo acima ilustra apenas a estrutura mínima de `raw_update`; na operação, deve ser persistido o **objeto integral** recebido da API Telegram, sem token. A relação opcional com `request` está em revisão: não definir duas FKs opostas sem decisão explícita. Consulte [request](request.md), [cursor](transport-cursor.md) e [pendências](../pendencias.md).

## Chaves, índices e integridade física

- **PK:** `inbound_update_id`; **UNIQUE:** `(client_id, transport, transport_update_id)`.
- **CHECK proposto:** `processing_status IN ('accepted','processing','completed','ignored','failed')`.
- **Índice proposto:** `(processing_status, accepted_at)` para retomada; `raw_update` deve ser acessível apenas ao serviço.

**Estado do DDL:** proposta de tabela; nenhum SQL de migração foi executado.

**Situação documental:** ficha revisada no A-Bioma Doc Standard; `refinement` é o estado de refinamento original, não prova de implementação. **Origem histórica:** [PST correspondente em `develop`](https://github.com/ahhvahh/a-bioma-sabia/blob/96427358088a6663176f0fb96de6b14166f975e4/docs/especificacao/persistencia/entidades/inbound-update.md).
