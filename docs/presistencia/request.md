# Entidade `request`

**ID:** PST-0103  
**Status:** refinement

## Objetivo

Persistir a requisição normalizada entregue ao Core, independente do formato específico do transporte.

## Dependências

- [CTR-0001 — Comando interno](../../contratos/comando-interno.md)
- [FLW-0001 — Comando Telegram](../../fluxos/comando-telegram.md)
- [PST-0102 — inbound_update](inbound-update.md)

## Estrutura e de-para propostos

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `request_id` | `uuid` | não | Sabiá | UUID v4 gerado na entrada | validar UUID v4; serializar externamente como string canônica |
| `inbound_update_id` | `bigint` | sim | PostgreSQL | PK de `inbound_update` | copiar FK quando a origem for update persistido |
| `client_id` | `text` | não | configuração/adaptador | cliente lógico que recebeu | copiar |
| `principal_id` | `text` | não | Telegram → adaptador | Telegram User ID | normalizar para string |
| `transport` | `text` | não | adaptador | `reply_context.transport` | copiar valor normalizado |
| `destination_id` | `text` | não | Telegram → adaptador | Chat ID | normalizar para string opaca |
| `source_message_id` | `text` | sim | Telegram | `message_id`, quando existir | normalizar para string |
| `command` | `text` | não | adaptador | comando identificado no update | token principal normalizado |
| `arguments` | `text[]` | não | adaptador | argumentos tokenizados | lista ordenada → `text[]`; sem argumentos = array vazio |
| `status` | `text` | não | Core/Sabiá | estado do processamento da requisição pelo Core | `received/processing/completed/failed` |
| `received_at` | `timestamptz` | não | adaptador | instante de aceite | normalizar para instante absoluto |
| `updated_at` | `timestamptz` | não | Sabiá | última alteração | relógio do serviço |

## Exemplo ilustrativo

| request_id | inbound_update_id | client_id | principal_id | transport | destination_id | source_message_id | command | arguments | status | received_at |
|---|---:|---|---|---|---|---|---|---|---|---|
| `550e8400-e29b-41d4-a716-446655440000` | 1201 | `bioma` | `778899` | `telegram` | `-100123` | `451` | `status` | `[]` | `completed` | `2026-10-02T23:30:01Z` |

## Identidade da request

O Sabiá gera `request_id` como **UUID v4** antes da primeira persistência da requisição.

Regras:

- a geração pertence ao Sabiá; adaptadores, processadores e produtores apenas propagam o valor;
- o PostgreSQL persiste `request_id` usando o tipo nativo `uuid`;
- contratos JSON e MessagePack representam o UUID como string canônica com hífens;
- o mesmo UUID é reutilizado em jobs, mensagens, mídia e transmissões da mesma correlação;
- colisão de chave não cria nova identidade e deve ser tratada como falha interna.

## Ciclo de vida de `request.status`

`request.status` representa exclusivamente o processamento da requisição pelo Core. Ele não representa a conclusão de jobs, o envio de mensagens nem a entrega de mídia.

Estados:

- `received` — requisição normalizada e persistida, ainda não entregue ao Core para processamento;
- `processing` — processamento pelo Core iniciado;
- `completed` — o Core terminou o processamento e produziu o resultado imediato esperado;
- `failed` — o Core encerrou o processamento com erro controlado ou falha que impede produzir resultado válido.

Transições permitidas:

`received → processing → completed | failed`

Para uma operação assíncrona, a criação e persistência bem-sucedida do job é o resultado imediato do Core. Portanto a requisição passa para `completed` após o job ser criado; o ciclo posterior do job permanece em `job.status`.

Da mesma forma, `request.status = completed` não significa que uma mensagem ou mídia foi entregue ao cliente. Entregas usam seus próprios estados em `outbound_message` e `media_transmission`.

Estados terminais de request são `completed` e `failed`.

## Elegibilidade para limpeza

O `request_id` existe para correlacionar o estado operacional enquanto a mensagem/tarefa está sendo administrada pelo Sabiá.

Quando o processamento associado termina, a correlação entra no processo de limpeza. Isso significa tornar o conjunto elegível para limpeza; não autoriza remover dependências ainda necessárias.

A remoção efetiva só pode ocorrer quando não existirem jobs em execução, mensagens pendentes de entrega ou transmissões de mídia pendentes/em andamento associadas ao `request_id`.

O prazo de retenção dos metadados concluídos não é definido por esta entidade.

## Origem Telegram

Para comandos recebidos por `Update.message`:

- `Update.message.from.id → principal_id`;
- `Update.message.chat.id → destination_id`;
- `Update.message.message_id → source_message_id`;
- `Update.message.text` é a fonte do comando e dos argumentos;
- `Update.message.entities[]` com `type = bot_command` identifica a entidade de comando.

`Update.update_id` pertence a `inbound_update.transport_update_id`, não à entidade `request`.

Esses caminhos já estão documentados em [Telegram discovery](../../../telegram-discovery.md) e consolidados por CTR-0010.

## Consumo

A entidade é a fonte de correlação para:

- criação de jobs;
- ingestão de mídia por socket usando `request_id`;
- construção de transmissões e respostas;
- autorização de visibilidade de jobs.
