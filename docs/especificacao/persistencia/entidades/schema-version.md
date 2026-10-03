# Entidade `schema_version`

**ID:** PST-0111  
**Status:** refinement

## Objetivo

Representar o versionamento do schema somente se o mecanismo de migração escolhido não possuir estrutura equivalente.

## Dependências

- [CFG-0001 — Modelo de configuração](../../configuracao/modelo-configuracao.md)
- [ADR-0009 — PostgreSQL](../../../adr/persistencia/estado-operacional.md)

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
