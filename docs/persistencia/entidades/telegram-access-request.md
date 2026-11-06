<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Entidade proposta `sabia.telegram_access_request`

> **Conteúdo lido e migrado para:** [ENT-013 — `sabia.telegram_access_request`](ENT-013-telegram-access-request.md). O identificador histórico e as decisões pendentes permanecem preservados neste original.

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

**ID:** PST-P02 · **Origem:** requisito funcional do projeto sem tabela física aprovada · **Situação:** proposta nova para decisão.

## Finalidade

Guardar pedidos de autorização por bot e a decisão do operador.

## Campos e tipos propostos

| Campo | Tipo PostgreSQL | Nulo | Origem/regra |
|---|---|---|---|
| `access_request_id` | `bigint identity` | não | PK gerada |
| `client_id` | `text` | não | Bot solicitado |
| `principal_id` | `text` | não | Telegram User ID |
| `status` | `text` | não | pending/approved/rejected |
| `requested_at` | `timestamptz` | não | Data do pedido |
| `decided_at` | `timestamptz` | sim | Data da decisão |
| `decided_by` | `text` | sim | Identidade Linux/operador |
| `decision_reason` | `text` | sim | Justificativa controlada |
| `updated_at` | `timestamptz` | não | Última transição |

## Chaves, índices e relacionamento propostos

- **PK:** `access_request_id`.
- **Unicidade:** Índice UNIQUE parcial `(client_id, principal_id) WHERE status = 'pending'` para evitar múltiplos pedidos abertos (proposto).
- **FK:** `(client_id, principal_id) → telegram_contact(client_id, principal_id) ON DELETE RESTRICT` (proposto).
- **Índice de consulta:** `(client_id, status, requested_at)` para fila administrativa.
- **Status/transições:** usar domínio `CHECK` quando indicado, sujeito à aprovação dos estados.

## Limites da especificação

Confirmar Y/N é regra de interface; decisão precisa ser auditável e atômica. Política de reenvio depois da rejeição ainda pendente.

**Não executar migração SQL com base nesta proposta antes da aprovação do modelo e da compatibilidade com YAML e contratos administrativos.** Ver [pendências](../pendencias.md) e [índice](README.md).
