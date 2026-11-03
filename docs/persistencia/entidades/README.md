# Entidades persistentes

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

Cada ficha define **nome da tabela qualificado pelo schema `sabia`**, campos, tipos PostgreSQL, nulabilidade, origem dos dados e regras de integridade.

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
| PST-0111 | [`sabia.schema_version`](schema-version.md) | refinement |

## Cinco propostas administrativas, ainda não aprovadas

| ID | Tabela sugerida |
|---|---|
| PST-P01 | [`sabia.telegram_contact`](telegram-contact.md) |
| PST-P02 | [`sabia.telegram_access_request`](telegram-access-request.md) |
| PST-P03 | [`sabia.telegram_authorization`](telegram-authorization.md) |
| PST-P04 | [`sabia.script_registry`](script-registry.md) |
| PST-P05 | [`sabia.action_catalog`](action-catalog.md) |

As fichas propostas **não representam schema existente**. Não criar tabelas de assinatura de eventos, logging, processamento de arquivos administrativos ou versão do schema sem os contratos correspondentes.

**Documentos de apoio:** [visão geral](../readme.md), [configuração SQL](../configuracao-banco.md), [relacionamentos](../relacionamentos.md), [pendências](../pendencias.md).
