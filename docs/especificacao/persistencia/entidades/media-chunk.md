# Entidade `media_chunk`

**ID:** PST-0109  
**Status:** refinement

## Objetivo

Persistir cada pedaço de uma mídia recebida pelo canal fracionado.

## Dependências

- [CTR-0009 — Ingestão fracionada](../../contratos/ingestao-midia-fracionada.md)
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
