# Origem e conversão dos dados persistentes

![CTR](https://img.shields.io/badge/CTR-CTR--0010-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir como informações provenientes de transportes, sockets, configuração e componentes internos são normalizadas antes de serem gravadas nas entidades PostgreSQL.

Este contrato é o de-para entre **dado de origem** e **campo persistido**. O detalhe por coluna permanece nos documentos de entidade.

## Dependências

- [PST-0100 — Entidades persistentes](../persistencia/entidades/README.md)
- [CTR-0001 — Comando interno](comando-interno.md)
- [CTR-0008 — Ingestão MessagePack](ingestao-midia-messagepack.md)
- [CTR-0009 — Ingestão fracionada](ingestao-midia-fracionada.md)
- [FLW-0001 — Comando Telegram](../fluxos/comando-telegram.md)

## Classes de origem

### Telegram

Para comandos recebidos em `Update.message`, o de-para bruto já documentado em [Telegram discovery](../../telegram-discovery.md) é:

| Telegram Bot API | Campo normalizado/persistido |
|---|---|
| `Update.update_id` | `inbound_update.transport_update_id` |
| `Update.message.from.id` | `request.principal_id` |
| `Update.message.chat.id` | `request.destination_id` |
| `Update.message.message_id` | `request.source_message_id` |
| `Update.message.text` | fonte para comando e argumentos |
| `Update.message.entities[]` com `type = bot_command` | identificação do comando dentro do texto |

Conversões normativas:

- User ID → `principal_id: text`;
- Chat ID → `destination_id: text`;
- `update_id` → `transport_update_id: bigint`;
- `message_id` → `source_message_id: text`;
- cliente configurado → `client_id: text`;
- transporte Telegram → `transport = "telegram"`;
- argumentos tokenizados a partir de `message.text` → `text[]`;
- resposta remota `Message.message_id` → `remote_message_id: text`.

Para mídia recebida, a descoberta existente confirma:

- `Update.message.photo[].file_id`;
- `Update.message.video.file_id`;
- `Update.message.document.file_id`;
- `getFile(file_id) → File.file_path`;
- download do conteúdo pelo endpoint de arquivo usando `file_path`.

A seleção normativa entre tamanhos de foto e o de-para final de nome/`content_type` para cada tipo de mídia ainda não estão fechados.

### Unix socket — mídia integral CTR-0008

De-para:

| MessagePack | Entidade/campo |
|---|---|
| `request_id` | `media.request_id` |
| `name` | `media.name` |
| `content_type` | `media.content_type` |
| `data` | `media.data` |
| `len(data)` | `media.size_bytes`, `media.total_bytes`, `media.received_bytes` |
| canal CTR-0008 | `media.storage_mode = inline` |
| commit do BLOB | `media.completed = true` |

`client_id`, `transport` e `destination_id` **não vêm do socket**. São recuperados pela `request_id` persistida e usados para criar `media_transmission`.

### Unix socket — abertura fracionada CTR-0009

| MessagePack | Entidade/campo |
|---|---|
| `request_id` | `media.request_id` |
| `name` | `media.name` |
| `content_type` | `media.content_type` |
| `total_bytes` | `media.total_bytes` |
| canal CTR-0009 | `media.storage_mode = chunked` |
| valor inicial | `received_bytes = 0` |
| valor inicial | `next_sequence_id = 1` |
| valor inicial | `completed = false` |

### Unix socket — chunk CTR-0009

| MessagePack | Entidade/campo |
|---|---|
| `media_id` | `media_chunk.media_id` |
| `sequence_id` | `media_chunk.sequence_id` |
| `data` | `media_chunk.data` |
| `len(data)` | `media_chunk.size_bytes` |
| soma após commit | `media.received_bytes` |
| sequência seguinte | `media.next_sequence_id` |
| comparação com total | `media.completed` |

### Normalização de `content_type`

Para qualquer origem de mídia:

- ausência ou `null` permanece `NULL`;
- string vazia ou somente com espaços é convertida para `NULL`;
- valor informado deve possuir sintaxe válida de media type;
- valor válido é normalizado para minúsculas antes da persistência;
- valor inválido produz `invalid_content_type`;
- o Sabiá não deriva o valor por extensão do nome nem por inspeção do conteúdo binário no MVP.

### Configuração YAML

Dados persistidos por referência lógica:

- cliente configurado → `client_id`;
- identificador do schedule → `alert_state.schedule_id`.

Configuração completa continua no YAML e não é copiada para tabelas apenas por conveniência.

### Estado interno do Sabiá

`request.status` usa o domínio `received | processing | completed | failed` e representa somente o processamento pelo Core. Estado de job e de entrega permanece nas respectivas entidades.


São gerados internamente:

- IDs identity do PostgreSQL;
- `request_id` UUID v4 gerado pelo Sabiá;
- estados de request/job/entrega;
- timestamps operacionais;
- `storage_mode`;
- contadores derivados;
- histórico de jobs;
- estado normalizado de alertas.

Cada valor derivado deve possuir regra explícita no documento da entidade correspondente.

## Regras de conversão

- IDs externos que o Core trata como opacos são persistidos como `text`;
- `request_id` é persistido como PostgreSQL `uuid` e representado nos contratos externos como string UUID canônica;
- timestamps persistidos usam `timestamptz`;
- binário MessagePack usa `bytea`;
- contadores de bytes usam `bigint`;
- lista ordenada de argumentos usa `text[]`;
- nenhuma origem pode preencher campo que o contrato declara como derivado do Sabiá;
- o produtor de mídia não pode fornecer `client_id`, `transport` ou `destination_id`.

## Saída do contrato

O resultado da conversão é um conjunto de valores normalizados apto a ser entregue ao fluxo FLW-0009.

## Erros

- campo obrigatório ausente;
- tipo incompatível;
- ID de correlação inexistente;
- valor fora do domínio;
- sequência inválida;
- tamanho excedido;
- origem tentando definir campo reservado ao Sabiá.

## BLOCKED

- seleção normativa do item de `Update.message.photo[]` quando houver múltiplos tamanhos;
- de-para final de nome e `content_type` para mídia recebida do Telegram.

## Critérios de aceite

- cada coluna persistente possui origem ou regra de derivação rastreável;
- socket não pode substituir dados de destino recuperados pela request;
- Telegram é normalizado antes de alcançar o Core;
- conversões não dependem de objetos específicos da Bot API depois da fronteira do adaptador.
