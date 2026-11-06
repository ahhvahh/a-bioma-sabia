<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-015 — Entidade `sabia.script_registry`

**ID documental:** `ENT-015` · **ID de origem:** `PST-P04`  
**Banco/schema:** `abioma_sabia` / `sabia` (nomes propostos).  
**Situação:** proposta física `refinement`, sem tabela aprovada ou migração executada.

## Finalidade

Catalogar scripts submetidos pelo operador, validade e aprovação antes da associação a ações.

## Campos e tipos propostos

| Campo | Tipo PostgreSQL | Nulo | Origem/regra |
|---|---|---|---|
| `script_id` | `text` | não | Identificador lógico |
| `description` | `text` | sim | Descrição de uso |
| `interpreter` | `text` | não | Interpretador permitido |
| `execution_mode` | `text` | não | conventional/async |
| `sha256` | `char(64)` | não | Integridade do conteúdo |
| `storage_ref` | `text` | não | Referência interna controlada, nunca caminho do usuário remoto |
| `status` | `text` | não | quarantined/validated/approved/rejected |
| `created_at` | `timestamptz` | não | Cadastro |
| `validated_at` | `timestamptz` | sim | Validação |
| `approved_at` | `timestamptz` | sim | Ativação autorizada |
| `approved_by` | `text` | sim | Operador responsável |

## Chaves, índices e relacionamento propostos

- **PK:** `script_id`.
- **Unicidade:** PK textual; política de versionamento ainda pendente.
- **FK:** Nenhuma FK externa obrigatória no modelo provisório.
- **Índice de consulta:** Índice `(status, created_at)` para triagem administrativa.
- **Status/transições:** usar domínio `CHECK` quando indicado, sujeito à aprovação dos estados.

## Limites da especificação

Conteúdo binário do script não é definido aqui; armazenamento e política de teste/versões ainda precisam de decisão. Cadastro não executa o arquivo.

**Não executar migração SQL com base nesta proposta antes da aprovação do modelo e da compatibilidade com YAML e contratos administrativos.** Ver [pendências](../pendencias.md) e [índice](README.md).

## Limites da revisão

**Pendente:** o armazenamento físico, versionamento dos scripts, avaliação em sandbox e sincronização com o cadastro YAML precisam de contrato administrativo. Os status da tabela são proposta, não transições aprovadas.

A ficha documenta campos, tipos PostgreSQL, nulabilidade, índices e relações **propostos**. Não aprova DDL, chaves adicionais nem migrações. Consulte [pendências físicas](../pendencias.md), [relacionamentos](../PERS-001-relacionamentos.md) e [índice das entidades](README.md).

## Rastreabilidade

Fonte histórica integral: [`PST-P04` — `sabia.script_registry`](script-registry.md). Nenhum campo do quadro de origem foi removido nesta revisão.
