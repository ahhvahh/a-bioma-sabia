# Entidade `outbound_message`

**ID:** PST-0106  
**Status:** refinement

## Objetivo

Persistir respostas textuais até confirmação de entrega pelo transporte.

## Dependências

- [CTR-0001 — Comando interno](../../contratos/comando-interno.md)
- [MOD-0002 — Adaptador Telegram](../../modulos/telegram.md)
- [FLW-0001 — Comando Telegram](../../fluxos/comando-telegram.md)
- [PST-0003 — Claim concorrente de filas persistentes](../claim-concorrente-filas.md)

## Estrutura e de-para propostos

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `outbound_message_id` | `bigint identity` | não | PostgreSQL | gerado | identity |
| `request_id` | `uuid` | não | resultado interno | `request_id` CTR-0001 | validar/copiar UUID v4 |
| `client_id` | `text` | não | `request` | `client_id` | copiar |
| `transport` | `text` | não | `request` | `reply_context.transport` | copiar |
| `destination_id` | `text` | não | `request` | `reply_context.destination_id` | copiar |
| `content` | `text` | não | adaptador | texto final a entregar | copiar a apresentação textual produzida pelo adaptador |
| `status` | `text` | não | Sabiá | lifecycle de entrega | `pending/sending/delivered/failed` |
| `remote_message_id` | `text` | sim | Telegram | `message_id` da resposta de sucesso | normalizar para string |
| `available_at` | `timestamptz` | sim | Telegram/Sabiá | `retry_after` + instante atual | calcular próxima elegibilidade |
| `last_error_code` | `text` | sim | transporte | código de falha | normalizar código |
| `last_error_message` | `text` | sim | transporte | descrição da falha | texto controlado |
| `created_at` | `timestamptz` | não | Sabiá | criação | instante absoluto |
| `updated_at` | `timestamptz` | não | Sabiá | última alteração | instante absoluto |

## Exemplo ilustrativo

| outbound_message_id | request_id | client_id | transport | destination_id | content | status | remote_message_id |
|---:|---|---|---|---|---|---|---|
| 901 | `550e8400-e29b-41d4-a716-446655440000` | `bioma` | `telegram` | `-100123` | `Serviço operacional` | `delivered` | `452` |

## Registro

Antes do envio, criar a linha em `pending`. O entregador deve obter a mensagem exclusivamente pelo claim de PST-0003, realizando `pending → sending` antes da chamada ao transporte. Após sucesso remoto, persistir `remote_message_id` e `delivered` na mesma transação. Falha temporária ou ambígua retorna `sending → pending`; falha permanente realiza `sending → failed`.

## Consumo

Entregador busca mensagens `pending` elegíveis por `available_at` através de PST-0003. No restart, mensagens encontradas em `sending` retornam para `pending` antes da retomada dos entregadores.

## Integridade referencial

`outbound_message.request_id → request.request_id` usa `ON DELETE RESTRICT`.

Mensagens elegíveis são removidas explicitamente pelo processo de limpeza antes da `request`; a referência nunca é convertida para `NULL`.

## Formato persistente do conteúdo

`outbound_message` persiste somente **entregas textuais**. Portanto `content: text` é o formato normativo do MVP.

O envelope CTR-0001 não é persistido nesta coluna:

- resultado `message` é convertido pelo adaptador para texto;
- resultado `job` é convertido pelo adaptador para a mensagem textual de confirmação/referência;
- resultado `error` é convertido pelo adaptador para texto preservando a semântica do erro;
- resultado `file` não usa `outbound_message`; a entrega de arquivo pertence a `media_transmission`.

Recovery reenvia o texto já materializado em `content`; não precisa reconstruir o envelope original do Core.


## BLOCKED

### Alertas agendados

`request_id` é obrigatório neste modelo, porém FLW-0003 pode gerar notificações do Scheduler sem uma `request` de origem documentada.

Antes de refinar esta entidade é necessário decidir como mensagens de alerta são correlacionadas e persistidas. Não deve ser criada `request` sintética, tornar `request_id` nulo ou adicionar outra chave de correlação sem decisão explícita.
