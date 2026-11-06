<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Entidade `sabia.media_chunk`

> **Conteúdo lido e migrado para:** [ENT-009 — documento canônico](ENT-009-media-chunk.md). Esta ficha `PST` permanece como fonte histórica; o DDL segue não verificado.

![Tipo](https://img.shields.io/badge/Tipo-entidade-0550ae?style=flat-square) ![Status](https://img.shields.io/badge/Status-refined-2da44e?style=flat-square)

**Banco proposto:** `abioma_sabia` · **Schema proposto:** `sabia` · **Tabela:** `sabia.media_chunk`

**ID:** PST-0109  
**Status:** refined

## Objetivo

Persistir cada pedaço de uma mídia recebida pelo canal fracionado.

## Dependências

- [CTR-0009 — Ingestão fracionada](../../contratos/CONTR-009-ingestao-midia-fracionada.md)
- [PST-0108 — media](media.md)

## Estrutura e de-para

| Campo | PostgreSQL | Nulo | Origem MessagePack | Conversão |
|---|---|---:|---|---|
| `media_id` | `bigint` | não | `media_id` | validar existência e copiar |
| `sequence_id` | `bigint` | não | `sequence_id` | validar sequência e copiar |
| `data` | `bytea` | não | `data` | binary → `bytea` |
| `size_bytes` | `bigint` | não | derivado de `data` | `len(data)` |
| `created_at` | `timestamptz` | não | Sabiá | instante do commit |

**PK proposta:** `(media_id, sequence_id)`.

## Exemplo ilustrativo

| media_id | sequence_id | size_bytes | created_at | data |
|---:|---:|---:|---|---|
| 502 | 1 | 5000000 | `2026-10-02T23:40:00Z` | `<bytea 5 MB>` |
| 502 | 2 | 5000000 | `2026-10-02T23:40:02Z` | `<bytea 5 MB>` |

## Registro

Inserção do chunk e atualização de `media.received_bytes`, `next_sequence_id` e `completed` pertencem à mesma transação.

## Consumo

Na transmissão, ler por `media_id` em `sequence_id ASC` e produzir stream contínuo.

## Restrições

- `sequence_id` começa em 1;
- máximo de 5.000.000 bytes por chunk;
- duplicidade de sequência não cria nova linha.


## Integridade referencial

`media_chunk.media_id → media.media_id` usa `ON DELETE RESTRICT`.

Chunks são removidos explicitamente pelo fluxo de limpeza; a exclusão da mídia não remove chunks automaticamente.

## Verificação de consistência

A chave composta `(media_id, sequence_id)` determina unicidade e ordem; rejeitar parte repetida ou fora da sequência esperada. Inserção de `media_chunk`, atualização dos contadores da [mídia](media.md) e avanço de `next_sequence_id` devem pertencer à mesma transação. Consulte o [contrato de ingestão fracionada](../../contratos/CONTR-009-ingestao-midia-fracionada.md).

## Chaves, índices e integridade física

- **PK:** `(media_id, sequence_id)`; **FK:** `media_id → media.media_id ON DELETE RESTRICT`.
- **CHECKs propostos:** `sequence_id >= 1`, `size_bytes BETWEEN 1 AND 5000000`, `octet_length(data) = size_bytes`.
- **Acesso:** leitura por `(media_id, sequence_id ASC)` usa a ordem da PK, sem índice duplicado.

**Estado do DDL:** proposta em revisão; nenhuma tabela foi criada.

**Situação documental:** ficha revisada no A-Bioma Doc Standard; `refined` é o estado de refinamento original, não prova de implementação. **Origem histórica:** [PST correspondente em `develop`](https://github.com/ahhvahh/a-bioma-sabia/blob/96427358088a6663176f0fb96de6b14166f975e4/docs/especificacao/persistencia/entidades/media-chunk.md).
