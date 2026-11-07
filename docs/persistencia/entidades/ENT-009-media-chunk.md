<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-009 — Entidade `sabia.media_chunk`

**ID canônico:** `ENT-009` · **ID histórico:** `PST-0109`  
**Banco e schema:** `abioma_sabia` / `sabia` (**propostos**)  
**Estado documental de origem:** `refined` · **Estado do DDL:** proposta; não há migração executada verificada.

## Objetivo

Persistir cada pedaço de uma mídia recebida pelo canal fracionado.

## Dependências

- [CONTR-009 (CTR-0009) — Ingestão fracionada](../../contratos/CONTR-009-ingestao-midia-fracionada.md)
- [ENT-008 (PST-0108) — media](ENT-008-media.md)

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

A chave composta `(media_id, sequence_id)` determina unicidade e ordem; rejeitar parte repetida ou fora da sequência esperada. Inserção de `media_chunk`, atualização dos contadores da [mídia](ENT-008-media.md) e avanço de `next_sequence_id` devem pertencer à mesma transação. Consulte o [contrato de ingestão fracionada](../../contratos/CONTR-009-ingestao-midia-fracionada.md).

## Chaves, índices e integridade física

- **PK:** `(media_id, sequence_id)`; **FK:** `media_id → media.media_id ON DELETE RESTRICT`.
- **CHECKs propostos:** `sequence_id >= 1`, `size_bytes BETWEEN 1 AND 5000000`, `octet_length(data) = size_bytes`.
- **Acesso:** leitura por `(media_id, sequence_id ASC)` usa a ordem da PK, sem índice duplicado.

**Estado do DDL:** proposta em revisão; nenhuma tabela foi criada.

## Revisão de consistência e pendências

**Integridade de chunks:** sequência e atualização de contadores pertencem à mesma transação. O identificador composto mantém ordem; não assumir migração SQL disponível.

Todas as decisões físicas pendentes permanecem registradas em [pendências PostgreSQL](../PERS-002-pendencias-modelo-fisico.md). A ficha descreve uma **especificação documental**, não comprova instalação do banco nem aprovação do DDL.

## Rastreabilidade

Fonte imediatamente anterior: [PST-0109 — `sabia.media_chunk`](https://github.com/ahhvahh/a-bioma-sabia/blob/d2fdacd8f48253e465d79127d298a1274d6f1269/docs/persistencia/entidades/media-chunk.md). A fonte histórica em `develop`, seus exemplos e os campos originais permanecem rastreáveis no arquivo legado.
