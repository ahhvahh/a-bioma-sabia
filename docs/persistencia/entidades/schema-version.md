<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Entidade `sabia.schema_version`

> **Conteúdo lido e migrado para:** [ENT-011 — `sabia.schema_version`](ENT-011-schema-version.md). O identificador histórico e as decisões pendentes permanecem preservados neste original.

![Tipo](https://img.shields.io/badge/Tipo-entidade-0550ae?style=flat-square) ![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

**Banco proposto:** `abioma_sabia` · **Schema proposto:** `sabia` · **Tabela:** `sabia.schema_version`

**ID:** PST-0111  
**Status:** refinement

## Objetivo

Representar o versionamento do schema somente se o mecanismo de migração escolhido não possuir estrutura equivalente.

## Dependências

- [CFG-0001 — Modelo de configuração](../../especificacao/configuracao/modelo-configuracao.md)
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
