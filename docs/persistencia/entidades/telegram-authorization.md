# Entidade proposta `sabia.telegram_authorization`

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

**ID:** PST-P03 · **Origem:** requisito funcional do projeto sem tabela física aprovada · **Situação:** proposta nova para decisão.

## Finalidade

Persistir autorização efetiva por cliente e identidade Telegram.

## Campos e tipos propostos

| Campo | Tipo PostgreSQL | Nulo | Origem/regra |
|---|---|---|---|
| `client_id` | `text` | não | Bot isolado |
| `principal_id` | `text` | não | Usuário Telegram |
| `active` | `boolean` | não | Autorização ativa |
| `granted_at` | `timestamptz` | não | Concessão |
| `granted_by` | `text` | não | Operador |
| `revoked_at` | `timestamptz` | sim | Revogação |
| `updated_at` | `timestamptz` | não | Última decisão |

## Chaves, índices e relacionamento propostos

- **PK:** `(client_id, principal_id)`.
- **Unicidade:** Chave composta garante isolamento por bot.
- **FK:** `(client_id, principal_id) → telegram_contact(client_id, principal_id) ON DELETE RESTRICT` (proposto).
- **Índice de consulta:** Índice `(client_id, active)` para consulta de autorizados.
- **Status/transições:** usar domínio `CHECK` quando indicado, sujeito à aprovação dos estados.

## Limites da especificação

Não inferir autorização pela existência do contato. Reconciliar precedência com `telegram.clients.allowed_users` do YAML antes de implantar.

**Não executar migração SQL com base nesta proposta antes da aprovação do modelo e da compatibilidade com YAML e contratos administrativos.** Ver [pendências](../pendencias.md) e [índice](README.md).
