<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-014 — Entidade `sabia.telegram_authorization`

**ID documental:** `ENT-014` · **ID de origem:** `PST-P03`  
**Banco/schema:** `abioma_sabia` / `sabia` (nomes propostos).  
**Situação:** proposta física `refinement`, sem tabela aprovada ou migração executada.

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

## Limites da revisão

**Pendente:** nenhuma autorização pode ser inferida pelo cadastro do contato. A precedência entre `telegram.clients.allowed_users` (YAML) e autorizações persistentes ainda não foi aprovada.

A ficha documenta campos, tipos PostgreSQL, nulabilidade, índices e relações **propostos**. Não aprova DDL, chaves adicionais nem migrações. Consulte [pendências físicas](../pendencias.md), [relacionamentos](../PERS-001-relacionamentos.md) e [índice das entidades](README.md).

## Rastreabilidade

Fonte histórica integral: [`PST-P03` — `sabia.telegram_authorization`](telegram-authorization.md). Nenhum campo do quadro de origem foi removido nesta revisão.
