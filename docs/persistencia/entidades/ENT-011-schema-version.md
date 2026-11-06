<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-011 — Entidade `sabia.schema_version`

**ID documental:** `ENT-011` · **ID de origem:** `PST-0111`  
**Banco/schema:** `abioma_sabia` / `sabia` (nomes propostos).  
**Situação:** tabela condicional, `refinement`; nenhum DDL ou migração executado.

## Objetivo

Representar o versionamento do schema somente se o mecanismo de migração escolhido não possuir estrutura equivalente.

## Dependências

- [CFG-0001 — Modelo de configuração](../../referencia/REF-001-modelo-configuracao.md)
- [ADR-0009 — PostgreSQL](../../decisoes/DEC-009-postgresql-estado-operacional.md)

## Estrutura proposta

| Campo | PostgreSQL | Nulo | Origem | Conversão |
|---|---|---:|---|---|
| `version` | `bigint` | não | migração | número/versionamento do mecanismo escolhido |
| `applied_at` | `timestamptz` | não | migrador | instante de conclusão |
| `description` | `text` | sim | migração | identificação textual |

## Exemplo ilustrativo

| version | applied_at | description |
|---:|---|---|
| 1 | `2026-10-02T23:00:00Z` | `initial_operational_schema` |

## Consumo

No startup, a versão persistida deve permitir validar/aplicar migrações compatíveis antes de liberar os demais módulos.

## BLOCKED

O mecanismo concreto de migração não está definido. Se a ferramenta adotada já mantiver tabela própria, esta entidade deve ser cancelada para evitar duas fontes de verdade.

## Chaves, índices e integridade física

- **PK proposta:** `version` (`bigint`).
- **Tabela condicional:** criar somente se a ferramenta de migração não oferecer controle equivalente.
- **Sem FK operacional.**

**Estado do DDL:** proposta em revisão; nenhuma tabela foi criada.

## Limites da revisão

**Decisão condicional:** a criação desta tabela depende da escolha do migrador. Se ele mantiver tabela própria de versionamento, a proposta deve ser descartada para impedir duas fontes de verdade. Não há migração executada ou mecanismo escolhido.

A ficha documenta campos, tipos PostgreSQL, nulabilidade, índices e relações **propostos**. Não aprova DDL, chaves adicionais nem migrações. Consulte [pendências físicas](../pendencias.md), [relacionamentos](../PERS-001-relacionamentos.md) e [índice das entidades](README.md).

## Rastreabilidade

Fonte histórica integral: [`PST-0111` — `sabia.schema_version`](schema-version.md). Nenhum campo do quadro de origem foi removido nesta revisão.
