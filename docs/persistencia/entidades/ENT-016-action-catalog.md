<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-016 — Entidade `sabia.action_catalog`

**ID documental:** `ENT-016` · **ID de origem:** `PST-P05`  
**Banco/schema:** `abioma_sabia` / `sabia` (nomes propostos).  
**Situação:** proposta física `refinement`, sem tabela aprovada ou migração executada.

## Finalidade

Representar ações e caminhos de menu específicos de cada bot, vinculados a scripts aprovados.

## Campos e tipos propostos

| Campo | Tipo PostgreSQL | Nulo | Origem/regra |
|---|---|---|---|
| `action_id` | `bigint identity` | não | PK gerada |
| `client_id` | `text` | não | Cliente Telegram |
| `logical_path` | `text` | não | Caminho hierárquico da ação |
| `label` | `text` | não | Nome mostrado no menu |
| `description` | `text` | sim | Descrição curta |
| `script_id` | `text` | não | Script autorizado |
| `execution_mode` | `text` | não | conventional/async |
| `enabled` | `boolean` | não | Publicação ativa |
| `created_at` | `timestamptz` | não | Cadastro |
| `updated_at` | `timestamptz` | não | Última mudança |

## Chaves, índices e relacionamento propostos

- **PK:** `action_id`.
- **Unicidade:** UNIQUE `(client_id, logical_path)`; menu derivado do caminho lógico.
- **FK:** `script_id → script_registry(script_id) ON DELETE RESTRICT` (proposto).
- **Índice de consulta:** Índice `(client_id, enabled, logical_path)` para montar menu.
- **Status/transições:** usar domínio `CHECK` quando indicado, sujeito à aprovação dos estados.

## Limites da especificação

Somente scripts aprovados e operações autorizadas podem ser executados; validade exige aplicação e checagem por cliente/bot. Permissões finas por ação ainda estão em definição.

**Não executar migração SQL com base nesta proposta antes da aprovação do modelo e da compatibilidade com YAML e contratos administrativos.** Ver [pendências](../pendencias.md) e [índice](README.md).

## Limites da revisão

**Pendente:** a relação com `script_registry` só é válida após aprovação do cadastro de scripts. Autorização fina por ação, publicação de menu e compatibilidade com YAML permanecem em definição.

A ficha documenta campos, tipos PostgreSQL, nulabilidade, índices e relações **propostos**. Não aprova DDL, chaves adicionais nem migrações. Consulte [pendências físicas](../pendencias.md), [relacionamentos](../PERS-001-relacionamentos.md) e [índice das entidades](README.md).

## Rastreabilidade

Fonte histórica integral: [`PST-P05` — `sabia.action_catalog`](action-catalog.md). Nenhum campo do quadro de origem foi removido nesta revisão.
