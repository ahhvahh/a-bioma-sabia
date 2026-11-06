<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-012 — Entidade `sabia.telegram_contact`

**ID documental:** `ENT-012` · **ID de origem:** `PST-P01`  
**Banco/schema:** `abioma_sabia` / `sabia` (nomes propostos).  
**Situação:** proposta física `refinement`, sem tabela aprovada ou migração executada.

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

**Não executar migração SQL com base nesta proposta antes da aprovação do modelo e da compatibilidade com YAML e contratos administrativos.** Ver [pendências](../PERS-002-pendencias-modelo-fisico.md) e [índice](README.md).

## Limites da revisão

**Limite de autorização:** registrar um contato em `/start` não concede acesso. O identificador de bot permanece lógico, sem tabela de clientes aprovada. Múltiplos destinos por contato e retenção seguem pendentes.

A ficha documenta campos, tipos PostgreSQL, nulabilidade, índices e relações **propostos**. Não aprova DDL, chaves adicionais nem migrações. Consulte [pendências físicas](../PERS-002-pendencias-modelo-fisico.md), [relacionamentos](../PERS-001-relacionamentos.md) e [índice das entidades](README.md).

## Rastreabilidade

Fonte histórica integral: [`PST-P01` — `sabia.telegram_contact`](telegram-contact.md). Nenhum campo do quadro de origem foi removido nesta revisão.
