<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-006 — Entidade `sabia.outbound_message`

**ID canônico:** `ENT-006` · **ID histórico:** `PST-0106`  
**Banco e schema:** `abioma_sabia` / `sabia` (**propostos**)  
**Estado documental de origem:** `refinement` · **Estado do DDL:** proposta; não há migração executada verificada.

## Objetivo

Persistir respostas textuais até confirmação de entrega pelo transporte.

## Dependências

- [CONTR-001 (CTR-0001) — Comando interno](../../contratos/CONTR-001-comando-interno.md)
- [ARQ-008 (MOD-0002) — Adaptador Telegram](../../arquitetura/ARQ-008-adaptador-telegram.md)
- [FLUX-001 (FLW-0001) — Comando Telegram](../../fluxos/FLUX-001-comando-telegram.md)
- [PST-0003 — Claim concorrente de filas persistentes](../concorrencia.md)

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

## Verificação de consistência

`delivered` exige confirmação do transporte registrada com `remote_message_id`; `sending` não prova entrega. Após falha ambígua e reinício, o reenvio pode ser repetido (semântica *at-least-once*). A geração de alertas sem `request_id` continua **BLOCKED**; nenhuma request sintética foi definida. Consulte [concorrência](../concorrencia.md) e [pendências](../PERS-002-pendencias-modelo-fisico.md).

## Chaves, índices e integridade física

- **PK:** `outbound_message_id`; **FK:** `request_id → request.request_id ON DELETE RESTRICT`.
- **CHECK proposto:** `status IN ('pending','sending','delivered','failed')`.
- **Índice proposto:** `(status, available_at, outbound_message_id)` para claim de envios; `request_id` deve ser indexado.

**Estado do DDL:** proposta de tabela; nenhum SQL de migração foi executado.

## Revisão de consistência e pendências

**Bloqueio DB-04:** mensagens do Scheduler sem `request_id` original ainda exigem correlação física. Não pressupor requisição sintética nem entrega garantida.

Todas as decisões físicas pendentes permanecem registradas em [pendências PostgreSQL](../PERS-002-pendencias-modelo-fisico.md). A ficha descreve uma **especificação documental**, não comprova instalação do banco nem aprovação do DDL.

## Rastreabilidade

Fonte imediatamente anterior: [PST-0106 — `sabia.outbound_message`](https://github.com/ahhvahh/a-bioma-sabia/blob/d2fdacd8f48253e465d79127d298a1274d6f1269/docs/persistencia/entidades/outbound-message.md). A fonte histórica em `develop`, seus exemplos e os campos originais permanecem rastreáveis no arquivo legado.
