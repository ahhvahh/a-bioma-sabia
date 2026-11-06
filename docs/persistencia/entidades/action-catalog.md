<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Entidade proposta `sabia.action_catalog`

> **Conteúdo lido e migrado para:** [ENT-016 — `sabia.action_catalog`](ENT-016-action-catalog.md). O identificador histórico e as decisões pendentes permanecem preservados neste original.

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

**ID:** PST-P05 · **Origem:** requisito funcional do projeto sem tabela física aprovada · **Situação:** proposta nova para decisão.

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
