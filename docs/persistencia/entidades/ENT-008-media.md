<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-008 — Entidade `sabia.media`

**ID canônico:** `ENT-008` · **ID histórico:** `PST-0108`  
**Banco e schema:** `abioma_sabia` / `sabia` (**propostos**)  
**Estado documental de origem:** `refinement` · **Estado do DDL:** proposta; não há migração executada verificada.

## Objetivo

Persistir a mídia lógica e seus metadados independentemente da origem ser socket local ou Telegram.

## Dependências

- [CONTR-006 (CTR-0006) — Mídia persistida](../../contratos/CONTR-006-midia-persistida.md)
- [CONTR-008 (CTR-0008) — Ingestão MessagePack](../../contratos/CONTR-008-ingestao-midia-messagepack.md)
- [CONTR-009 (CTR-0009) — Ingestão fracionada](../../contratos/CONTR-009-ingestao-midia-fracionada.md)
- [ENT-003 (PST-0103) — request](ENT-003-request.md)

## Estrutura proposta

| Campo | PostgreSQL | Nulo | Origem | Origem simples CTR-0008 | Origem fracionada CTR-0009 | Conversão |
|---|---|---:|---|---|---|---|
| `media_id` | `bigint identity` | não | PostgreSQL | — | — | identity |
| `request_id` | `uuid` | não | produtor/Telegram normalizado | `request_id` | `request_id` | validar UUID v4, existência e copiar |
| `name` | `text` | não | produtor/adaptador | `name` | `name` | copiar nome lógico |
| `content_type` | `text` | sim | produtor/adaptador | `content_type` | `content_type` | vazio → `NULL`; validar media type; normalizar para minúsculas |
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
| 502 | `550e8400-e29b-41d4-a716-446655440000` | `video.mp4` | `video/mp4` | 40000000 | 10000000 | `chunked` | 3 | `false` |

## Integridade referencial

`media.request_id → request.request_id` usa `ON DELETE RESTRICT`.

A remoção física de uma mídia exige remoção explícita prévia de dependências elegíveis, como chunks e transmissões. Não existe `ON DELETE CASCADE` no MVP.

## Origem Telegram

A documentação de descoberta já identifica as fontes de arquivo:

- foto: `Update.message.photo[].file_id`;
- vídeo: `Update.message.video.file_id`;
- documento: `Update.message.document.file_id`;
- resolução do conteúdo: `getFile(file_id) → File.file_path`, seguida do download do arquivo.

Ainda não está definido:

- qual item de `photo[]` é selecionado quando houver vários tamanhos;
- como obter/normalizar `name` para cada tipo de mídia;
- como preencher `content_type` quando o objeto Telegram não fornecer MIME explícito.

## BLOCKED

- seleção de foto e de-para final de `name`/`content_type` para mídia Telegram.

## Verificação de consistência

`size_bytes` e `received_bytes` descrevem conteúdo persistido, enquanto `total_bytes` representa o total esperado. Mídia `chunked` somente pode ser entregue com `completed = true`. Os limites documentados são **20.000.000 bytes** para ingestão simples e **50.000.000 bytes** para total fracionado; cada chunk aceita até **5.000.000 bytes**. Consulte [chunks](ENT-009-media-chunk.md) e [transmissões](ENT-010-media-transmission.md).

## Chaves, índices e integridade física

- **PK:** `media_id`; **FK:** `request_id → request.request_id ON DELETE RESTRICT`.
- **CHECKs propostos:** `storage_mode IN ('inline','chunked')`, `total_bytes >= 0`, `received_bytes BETWEEN 0 AND total_bytes`, `size_bytes >= 0`.
- **Índice proposto:** `(request_id, created_at)`; mídias incompletas não podem gerar transmissão.
- **Regra de tamanho:** canal simples até 20.000.000 bytes; canal fracionado até 50.000.000 bytes no total.

**Estado do DDL:** proposta em revisão; nenhuma tabela foi criada.

## Revisão de consistência e pendências

**Limites documentais:** ingestão integral até 20.000.000 bytes; fracionada até 50.000.000 bytes, com chunk até 5.000.000. Entrega fracionada somente após `completed = true`.

Todas as decisões físicas pendentes permanecem registradas em [pendências PostgreSQL](../PERS-002-pendencias-modelo-fisico.md). A ficha descreve uma **especificação documental**, não comprova instalação do banco nem aprovação do DDL.

## Rastreabilidade

Fonte imediatamente anterior: [PST-0108 — `sabia.media`](https://github.com/ahhvahh/a-bioma-sabia/blob/d2fdacd8f48253e465d79127d298a1274d6f1269/docs/persistencia/entidades/media.md). A fonte histórica em `develop`, seus exemplos e os campos originais permanecem rastreáveis no arquivo legado.
