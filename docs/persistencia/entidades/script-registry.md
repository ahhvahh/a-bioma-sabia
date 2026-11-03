# Entidade proposta `sabia.script_registry`

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

**ID:** PST-P04 · **Origem:** requisito funcional do projeto sem tabela física aprovada · **Situação:** proposta nova para decisão.

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
