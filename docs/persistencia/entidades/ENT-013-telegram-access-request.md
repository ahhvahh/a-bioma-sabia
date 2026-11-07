<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-013 — Entidade `sabia.telegram_access_request`

**ID documental:** `ENT-013` · **ID de origem:** `PST-P02`  
**Banco/schema:** `abioma_sabia` / `sabia` (nomes propostos).  
**Situação:** proposta física `refinement`, sem tabela aprovada ou migração executada.

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

**Não executar migração SQL com base nesta proposta antes da aprovação do modelo e da compatibilidade com YAML e contratos administrativos.** Ver [pendências](../PERS-002-pendencias-modelo-fisico.md) e [índice](README.md).

## Limites da revisão

**Pendente:** o índice UNIQUE parcial evita mais de um pedido `pending` por bot/usuário, mas reenvio após rejeição e política de auditoria precisam de decisão. A confirmação Y/N é um requisito de interface, não uma regra SQL implementada.

A ficha documenta campos, tipos PostgreSQL, nulabilidade, índices e relações **propostos**. Não aprova DDL, chaves adicionais nem migrações. Consulte [pendências físicas](../PERS-002-pendencias-modelo-fisico.md), [relacionamentos](../PERS-001-relacionamentos.md) e [índice das entidades](README.md).

## Rastreabilidade

Fonte histórica integral: [`PST-P02` — `sabia.telegram_access_request`](https://github.com/ahhvahh/a-bioma-sabia/blob/d2fdacd8f48253e465d79127d298a1274d6f1269/docs/persistencia/entidades/telegram-access-request.md). Nenhum campo do quadro de origem foi removido nesta revisão.
