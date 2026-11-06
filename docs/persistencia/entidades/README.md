# Entidades persistentes

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

Cada ficha define nome de tabela no schema **proposto** `sabia`, campos, tipos PostgreSQL, nulabilidade, origem dos dados e integridade documental. Nenhuma tabela ou migração foi verificada na branch. As propostas podem exigir alterações antes de serem aprovadas.

## Onze entidades herdadas do modelo de estado operacional

| ID | Entidade | Estado do documento de origem |
|---|---|---|
| PST-0101 | [`sabia.transport_cursor`](transport-cursor.md) | refined |
| PST-0102 | [`sabia.inbound_update`](inbound-update.md) | refinement |
| PST-0103 | [`sabia.request`](request.md) | refinement |
| PST-0104 | [`sabia.job`](job.md) | refined |
| PST-0105 | [`sabia.job_state_history`](job-state-history.md) | refinement |
| PST-0106 | [`sabia.outbound_message`](outbound-message.md) | refinement |
| PST-0107 | [`sabia.alert_state`](alert-state.md) | refinement |
| PST-0108 | [`sabia.media`](media.md) | refinement |
| PST-0109 | [`sabia.media_chunk`](media-chunk.md) | refined |
| PST-0110 | [`sabia.media_transmission`](media-transmission.md) | refinement |
| PST-0111 | [`sabia.schema_version`](ENT-011-schema-version.md) | refinement |

## Cinco propostas administrativas, ainda não aprovadas

| ID | Tabela sugerida |
|---|---|
| PST-P01 | [`sabia.telegram_contact`](ENT-012-telegram-contact.md) |
| PST-P02 | [`sabia.telegram_access_request`](ENT-013-telegram-access-request.md) |
| PST-P03 | [`sabia.telegram_authorization`](ENT-014-telegram-authorization.md) |
| PST-P04 | [`sabia.script_registry`](ENT-015-script-registry.md) |
| PST-P05 | [`sabia.action_catalog`](ENT-016-action-catalog.md) |

As fichas `ENT-012` a `ENT-016` são revisões documentais das propostas `PST-P01` a `PST-P05`; **não representam schema existente**, não aprovam DDL e preservam os arquivos históricos. `ENT-011` continua condicional à escolha de migrador. Não criar tabelas de assinatura de eventos, logging, processamento de arquivos administrativos ou versão do schema sem os contratos correspondentes.

**Fonte histórica:** as 6 fichas originais (sem prefixo `ENT`) permanecem no mesmo diretório, com links de retorno para os destinos canônicos.

**Documentos de apoio:** [visão geral](../readme.md), [configuração SQL](../configuracao-banco.md), [relacionamentos](../PERS-001-relacionamentos.md), [pendências](../pendencias.md).
