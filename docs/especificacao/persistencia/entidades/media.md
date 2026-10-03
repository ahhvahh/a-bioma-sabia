# Entidade `media`

**ID:** PST-0108  
**Status:** refinement

## Objetivo

Persistir a mídia lógica e seus metadados independentemente da origem ser socket local ou Telegram.

## Dependências

- [CTR-0006 — Mídia persistida](../../contratos/midia-persistida.md)
- [CTR-0008 — Ingestão MessagePack](../../contratos/ingestao-midia-messagepack.md)
- [CTR-0009 — Ingestão fracionada](../../contratos/ingestao-midia-fracionada.md)
- [PST-0103 — request](request.md)

## Estrutura proposta

| Campo | PostgreSQL | Nulo | Origem | Origem simples CTR-0008 | Origem fracionada CTR-0009 | Conversão |
|---|---|---:|---|---|---|---|
| `media_id` | `bigint identity` | não | PostgreSQL | — | — | identity |
| `request_id` | `uuid` | não | produtor/Telegram normalizado | `request_id` | `request_id` | validar UUID v4, existência e copiar |
| `name` | `text` | não | produtor/adaptador | `name` | `name` | copiar nome lógico |
| `content_type` | `text` | sim | produtor/adaptador | `content_type` | `content_type` | copiar; validação BLOCKED |
| `size_bytes` | `bigint` | não | derivado | `len(data)` | bytes persistidos acumulados | calcular |
| `total_bytes` | `bigint` | não | derivado/produtor | `len(data)` | `total_bytes` | converter para `bigint` |
| `received_bytes` | `bigint` | não | derivado | `len(data)` | inicia 0; acumula chunks | calcular atomicamente |
| `data` | `bytea` | sim | socket/adaptador | `data` | `NULL` | binary → `bytea` |
| `storage_mode` | `text` | não | Sabiá | `inline` | `chunked` | valor derivado do canal |
| `next_sequence_id` | `bigint` | sim | Sabiá | `NULL` | inicia `1` | incrementar após commit |
| `completed` | `boolean` | não | Sabiá | `true` após persistência | `received_bytes == total_bytes` | derivar |
| `created_at` | `timestamptz` | não | Sabiá | commit inicial | abertura | instante absoluto |

## Exemplo — upload integral

| media_id | request_id | name | content_type | total_bytes | received_bytes | storage_mode | next_sequence_id | completed |
|---:|---|---|---|---:|---:|---|---|---|
| 501 | `550e8400-e29b-41d4-a716-446655440000` | `foto.jpg` | `image/jpeg` | 145200 | 145200 | `inline` | `NULL` | `true` |

## Exemplo — upload fracionado em andamento

| media_id | request_id | name | content_type | total_bytes | received_bytes | storage_mode | next_sequence_id | completed |
|---:|---|---|---|---:|---:|---|---:|---|
| 502 | `550e8400-e29b-41d4-a716-446655440000` | `video.mp4` | `video/mp4` | 80000000 | 10000000 | `chunked` | 3 | `false` |

## Origem Telegram

A documentação define que mídia recebida pelo Telegram deve ser normalizada para CTR-0006, mas ainda não define o de-para bruto dos objetos Telegram para `name`, `content_type` e binário.

## BLOCKED

- regra normativa de `content_type`;
- de-para bruto da mídia Telegram para esta entidade.
