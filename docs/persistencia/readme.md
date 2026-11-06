# Persistência de dados — A-Bioma Sabiá

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

> **Modelo documental proposto.** Este repositório não comprova banco instalado nem migrações executáveis. Tabelas em `refinement` e modelos novos dependem de validação antes da implementação.

## Identificação e tecnologia

| Parâmetro | Valor | Situação |
|---|---|---|
| Tecnologia | **PostgreSQL**, SGBD relacional | Decisão existente (ADR-0009) |
| Banco lógico | `abioma_sabia` | **Nome proposto nesta reorganização** |
| Schema SQL | `sabia` | **Nome proposto nesta reorganização** |
| Role SQL de execução | `sabia` | **Nome proposto**; coincide com usuário Linux para autenticação `peer` |
| Role proprietária/migração | `sabia_owner` (NOLOGIN) | **Nome proposto** |
| Comunicação SQL | Socket Unix PostgreSQL restrito ao grupo Linux `abioma_db` | Requisito de arquitetura; configuração por instância |
| Clientes do serviço | Grupo Linux `absabia`, somente UDS do Sabiá | Não recebem SQL nem credenciais PostgreSQL |
| Arquivo de configuração | `/etc/sabia/sabia.yaml` | Configuração fora das tabelas SQL |
| Tipos | `uuid`, `bigint identity`, `text`, `text[]`, `timestamptz`, `jsonb`, `bytea`, `boolean` | Tipos usados pelas propostas existentes |

A versão mínima do PostgreSQL **não está fixada** na documentação. Os nomes de banco, schema e roles acima são escolhas **propostas**, não decisões pré-existentes.

## Organização

| Documento | Conteúdo |
|---|---|
| [Configuração do banco](configuracao-banco.md) | Banco, schema, roles, permissões e UDS, com exemplo de inicialização |
| [Relacionamentos](PERS-001-relacionamentos.md) | Diagrama Mermaid, PK, FK, cardinalidades e decisões em aberto |
| [Concorrência e filas](concorrencia.md) | Claim transacional, recuperação e ordem de processamento |
| [Entidades](entidades/README.md) | Campos, tipos, nulabilidade, índices e regras por tabela |
| [Pendências físicas](pendencias.md) | Restrições ainda não aprovadas para implementação |

## Entidades operacionais (origem documental PST)

| Tabela no schema `sabia` | Ficha |
|---|---|
| `transport_cursor` | [transport-cursor](entidades/transport-cursor.md) |
| `inbound_update` | [inbound-update](entidades/inbound-update.md) |
| `request` | [request](entidades/request.md) |
| `job` | [job](entidades/job.md) |
| `job_state_history` | [job-state-history](entidades/job-state-history.md) |
| `outbound_message` | [outbound-message](entidades/outbound-message.md) |
| `alert_state` | [alert-state](entidades/alert-state.md) |
| `media` | [media](entidades/media.md) |
| `media_chunk` | [media-chunk](entidades/media-chunk.md) |
| `media_transmission` | [media-transmission](entidades/media-transmission.md) |
| `schema_version` | [schema-version](entidades/ENT-011-schema-version.md) |

`schema_version` é **condicional**: não deve existir se o mecanismo de migração possuir tabela própria.

## Entidades adicionais requeridas pelo produto (a validar)

| Modelo proposto | Ficha |
|---|---|
| `telegram_contact` | [Contatos Telegram](entidades/ENT-012-telegram-contact.md) |
| `telegram_access_request` | [Solicitações de acesso](entidades/ENT-013-telegram-access-request.md) |
| `telegram_authorization` | [Autorização por bot](entidades/ENT-014-telegram-authorization.md) |
| `script_registry` | [Registro e validação de scripts](entidades/ENT-015-script-registry.md) |
| `action_catalog` | [Catálogo persistente de ações](entidades/ENT-016-action-catalog.md) |

Esses **cinco modelos são sugestões físicas**, pois o [documento de origem](https://github.com/ahhvahh/a-bioma-sabia/blob/e1858c007f88974bc7968d16f7092b645d8d9d18/docs/persistencia.md) registra essas capacidades como ainda não definidas em nível de tabelas. Não são entidades aprovadas ou existentes. Assinaturas por evento, auditoria detalhada e arquivos administrativos continuam pendentes de contrato físico.

## Regras de fronteira

- Somente o serviço `sabia` acessa diretamente o PostgreSQL, por UDS; `sabiactl` e outros consumidores usam apenas sockets da aplicação.
- `/start` registra contato, mas **não concede autorização**. Aprovações são confirmadas pelo operador e auditadas.
- Catálogos e autorizações devem ser persistentes, mas há divergência a resolver entre cadastro dinâmico e YAML estático.
- O Sabiá persiste estados, filas, mídia e correlações em PostgreSQL; `media.data` e `media_chunk.data` usam `bytea`.
- Todas as relações operacionais propostas usam `ON DELETE RESTRICT`; não há cascata automática e retenção/purge ainda precisa de decisão.
- **Não implementar** propostas com status `BLOCKED` sem resolver sua definição.

**Classificação:** as propostas `PST-P01` a `PST-P05` e a tabela condicional `PST-0111` possuem fichas canônicas `ENT-011` a `ENT-016`; os originais foram preservados.

**Referências internas:** [segurança](../arquitetura/ARQ-006-seguranca-autorizacao.md), [interfaces](../interfaces-servicos.md), [pendências gerais](../pendencias.md), [ADR-0009](../decisoes/DEC-009-postgresql-estado-operacional.md).
