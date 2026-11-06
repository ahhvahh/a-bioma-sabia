<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Entidade proposta `sabia.telegram_contact`

> **Conteúdo lido e migrado para:** [ENT-012 — `sabia.telegram_contact`](ENT-012-telegram-contact.md). O identificador histórico e as decisões pendentes permanecem preservados neste original.

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

**ID:** PST-P01 · **Origem:** requisito funcional do projeto sem tabela física aprovada · **Situação:** proposta nova para decisão.

## Finalidade

Registrar o contato reconhecido por `/start` sem aprová-lo automaticamente.

## Campos e tipos propostos

| Campo | Tipo PostgreSQL | Nulo | Origem/regra |
|---|---|---|---|
| `client_id` | `text` | não | Bot lógico (YAML) |
| `principal_id` | `text` | não | Telegram User ID normalizado |
| `last_destination_id` | `text` | não | Último Chat ID conhecido |
| `username` | `text` | sim | Nome público opcional |
| `display_name` | `text` | sim | Nome de apresentação opcional |
| `first_seen_at` | `timestamptz` | não | Primeiro contato |
| `last_seen_at` | `timestamptz` | não | Contato mais recente |

## Chaves, índices e relacionamento propostos

- **PK:** `(client_id, principal_id)`.
- **Unicidade:** Chave primária composta.
- **FK:** Sem FK física para cliente: bots ainda são configurados em YAML.
- **Índice de consulta:** Índice em `(client_id, last_seen_at DESC)` para consulta administrativa.
- **Status/transições:** usar domínio `CHECK` quando indicado, sujeito à aprovação dos estados.

## Limites da especificação

Um contato não implica autorização. A necessidade de guardar múltiplos destinos por contato ainda precisa ser decidida.

**Não executar migração SQL com base nesta proposta antes da aprovação do modelo e da compatibilidade com YAML e contratos administrativos.** Ver [pendências](../pendencias.md) e [índice](README.md).
