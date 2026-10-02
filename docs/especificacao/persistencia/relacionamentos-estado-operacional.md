# Relacionamentos do estado operacional

**ID:** PST-0002  
**Status:** refinement

## Objetivo

Representar os relacionamentos e cardinalidades propostos entre as tabelas de PST-0001, permitindo revisão do modelo antes da implementação.

Este documento não altera contratos existentes. Enquanto estiver em `refinement`, cardinalidades e políticas de integridade aqui descritas são propostas para aprovação.

## Dependências

- [ADR-0009 — Persistência do estado operacional em PostgreSQL](../../adr/persistencia/estado-operacional.md)
- [PST-0001 — Proposta de tabelas do estado operacional](tabelas-estado-operacional.md)
- [CTR-0003 — Job](../contratos/job.md)
- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [FLW-0001 — Comando Telegram](../fluxos/comando-telegram.md)
- [FLW-0003 — Monitoramento agendado](../fluxos/monitoramento-agendado.md)

## Visão geral

```mermaid
erDiagram
    TRANSPORT_CURSOR {
        text client_id PK
        text transport PK
        bigint last_update_id
        timestamptz updated_at
    }

    INBOUND_UPDATE {
        bigint inbound_update_id PK
        text client_id
        text transport
        bigint transport_update_id
        text request_id
        timestamptz accepted_at
    }

    REQUEST {
        text request_id PK
        bigint inbound_update_id
        text client_id
        text principal_id
        text transport
        text destination_id
        text command
        text status
        timestamptz received_at
    }

    JOB {
        bigint job_id PK
        text request_id
        text status
        text operation
    }

    JOB_STATE_HISTORY {
        bigint job_state_history_id PK
        bigint job_id
        text status
        timestamptz changed_at
    }

    OUTBOUND_MESSAGE {
        bigint outbound_message_id PK
        text request_id
        text status
        text remote_message_id
    }

    MEDIA {
        bigint media_id PK
        text request_id
        text storage_mode
        bigint total_bytes
        bigint received_bytes
        boolean completed
    }

    MEDIA_CHUNK {
        bigint media_id PK
        bigint sequence_id PK
        bytea data
    }

    MEDIA_TRANSMISSION {
        bigint transmission_id PK
        bigint media_id
        text request_id
        text status
    }

    ALERT_STATE {
        text schedule_id PK
        text state
        timestamptz last_notified_at
    }

    INBOUND_UPDATE o|--o| REQUEST : "origina"
    REQUEST ||--o{ JOB : "pode criar"
    JOB ||--|{ JOB_STATE_HISTORY : "registra estados"
    REQUEST ||--o{ OUTBOUND_MESSAGE : "produz"
    REQUEST ||--o{ MEDIA : "associa"
    MEDIA ||--o{ MEDIA_CHUNK : "possui quando chunked"
    MEDIA ||--o{ MEDIA_TRANSMISSION : "possui entregas"
    REQUEST ||--o{ MEDIA_TRANSMISSION : "correlaciona"
```

`TRANSPORT_CURSOR`, `ALERT_STATE` e `SCHEMA_VERSION` não precisam de FK entre si ou com as demais tabelas para atender os contratos atuais.

## Relações propostas

### `inbound_update → request`

**Cardinalidade proposta:** `0..1 : 0..1`.

Motivo:

- FLW-0001 exige persistir o update antes de avançar o offset;
- autorização ocorre depois desse aceite;
- um update persistido pode não produzir uma requisição executável;
- quando houver requisição, a correlação deve ser única.

Proposta:

- `request.inbound_update_id` como FK opcional;
- `UNIQUE(request.inbound_update_id)` quando não nulo;
- `inbound_update.request_id` é redundante se a FK existir em `request`.

**Ponto para revisão:** escolher somente uma direção física para evitar duplicação. A proposta preferida é manter a FK apenas em `request.inbound_update_id` e remover `inbound_update.request_id` de PST-0001 caso esta relação seja aprovada.

### `request → job`

**Cardinalidade proposta:** `1 : 0..N`.

Os contratos atuais não afirmam que uma requisição só pode criar um único job. Portanto a proposta não adiciona unicidade em `job.request_id`.

FK proposta:

`job.request_id → request.request_id`.

### `job → job_state_history`

**Cardinalidade proposta:** `1 : 1..N`.

Todo job deve possuir ao menos o estado inicial persistido.

FK proposta:

`job_state_history.job_id → job.job_id`.

Regra transacional proposta:

- criação do job e primeiro histórico na mesma transação;
- mudança de `job.status` e inclusão do histórico correspondente na mesma transação.

### `request → outbound_message`

**Cardinalidade proposta:** `1 : 0..N`.

Uma requisição pode produzir confirmação, progresso e resultado final. Não é proposta restrição de uma única mensagem por requisição.

FK proposta:

`outbound_message.request_id → request.request_id`.

### `request → media`

**Cardinalidade:** `1 : 0..N`.

Esta relação já é compatível com CTR-0006: uma mesma `request_id` pode possuir vários registros de mídia, inclusive com nome/conteúdo repetidos.

FK proposta:

`media.request_id → request.request_id`.

### `media → media_chunk`

**Cardinalidade:** `1 : 0..N`.

- `storage_mode = inline`: zero chunks;
- `storage_mode = chunked`: um ou mais chunks até completar `total_bytes`.

FK proposta:

`media_chunk.media_id → media.media_id`.

Chave de ordem:

`PRIMARY KEY (media_id, sequence_id)`.

A sequência e os contadores pertencem à mesma unidade transacional definida em CTR-0009.

### `media → media_transmission`

**Cardinalidade proposta:** `1 : 0..N`.

CTR-0007 define transmissões independentes e permite várias mídias por requisição. O modelo não deve pressupor que uma mídia jamais possa possuir mais de uma entrega.

FK proposta:

`media_transmission.media_id → media.media_id`.

### `request → media_transmission`

**Cardinalidade:** `1 : 0..N`.

CTR-0007 exige `request_id` dentro da transmissão para rastreabilidade.

FK proposta:

`media_transmission.request_id → request.request_id`.

A presença simultânea de `media_id` e `request_id` permite detectar inconsistência de correlação. Deve existir validação para impedir uma transmissão cujo `request_id` seja diferente do `request_id` da mídia.

**Ponto para revisão:** decidir se essa consistência será garantida apenas pela aplicação ou por uma restrição adicional no banco.

### `schedule configuração → alert_state`

**Cardinalidade conceitual:** `1 : 0..1`.

O agendamento pertence ao YAML de CFG-0001 e não é uma tabela proposta.

`alert_state.schedule_id` usa o identificador lógico da configuração, sem FK física.

Se um agendamento for removido da configuração, a política de retenção do `alert_state` correspondente ainda precisa ser decidida.

## Relações não propostas

### `job → media`

Não é necessária FK direta. A correlação existente por `request_id` atende os contratos atuais.

### `job → outbound_message`

Não é necessária FK direta para o MVP. O caminho de correlação proposto é:

`job.request_id → request.request_id → outbound_message.request_id`.

### `transport_cursor → inbound_update`

Não é necessária FK. O cursor representa estado agregado por `client_id + transport`, enquanto os updates mantêm a evidência idempotente individual.

### `schedule`

Não é proposta tabela de agendamentos enquanto CFG-0001 mantiver YAML como fonte operacional.

## Unidades transacionais propostas

### Aceite de update

Para Telegram:

1. inserir `inbound_update` respeitando a unicidade;
2. atualizar `transport_cursor.last_update_id`;
3. commit;
4. somente depois permitir o avanço do offset remoto.

Os passos 1 e 2 são propostos como uma única transação.

### Criação e mudança de estado de job

Criação:

1. inserir `job` em `queued`;
2. inserir `job_state_history` correspondente;
3. commit.

Transição:

1. validar transição conforme CTR-0003;
2. atualizar `job.status`;
3. inserir novo histórico;
4. commit.

### Entrega textual

Ao receber confirmação positiva do transporte:

1. persistir `remote_message_id`;
2. alterar `outbound_message.status` para `delivered`;
3. commit.

Nenhuma entrega deve ficar `delivered` sem confirmação persistida.

### Ingestão de mídia integral

Mídia e transmissão necessárias ao ACK do produtor devem respeitar a atomicidade já determinada por CTR-0007/CTR-0008.

### Ingestão de chunk

Para cada chunk aceito:

1. inserir `media_chunk`;
2. atualizar `media.received_bytes`;
3. atualizar `media.next_sequence_id`;
4. atualizar `media.completed` quando aplicável;
5. commit;
6. somente então enviar ACK.

## Política de exclusão proposta para revisão

Nenhuma política abaixo é aprovada enquanto PST-0002 estiver em `refinement`.

| Relação | Proposta inicial | Motivo |
|---|---|---|
| request → job | `RESTRICT` | preservar rastreabilidade operacional |
| job → job_state_history | `RESTRICT` | não apagar histórico implicitamente |
| request → outbound_message | `RESTRICT` | preservar entregas e recovery |
| request → media | `RESTRICT` | mídia possui retenção própria |
| media → media_chunk | exclusão explícita pelo fluxo de limpeza | FLW-0008 controla quando chunks podem ser removidos |
| media → media_transmission | `RESTRICT` | não perder histórico de entrega |
| request → media_transmission | `RESTRICT` | manter correlação |

Não é proposta exclusão em cascata automática de dados operacionais históricos no MVP.

## Pontos para revisão

Antes de `refined`, decidir:

- direção física final da relação `inbound_update/request`;
- se uma requisição pode criar mais de um job;
- garantia de consistência entre `media.request_id` e `media_transmission.request_id`;
- política de retenção de `alert_state` após remoção de um schedule;
- políticas finais de `ON DELETE`;
- se haverá retenção/purge de requests, jobs e mensagens finalizadas;
- relacionamento futuro de `job_attempt` caso retry automático seja habilitado.

## Critérios de aceite

- cada relação possui cardinalidade explícita;
- relações já determinadas pelos contratos são preservadas;
- relações propostas são distinguíveis de decisões já aprovadas;
- nenhuma FK exige tabela de configuração inexistente;
- unidades transacionais críticas são identificadas;
- políticas de exclusão permanecem em revisão até aprovação explícita.
