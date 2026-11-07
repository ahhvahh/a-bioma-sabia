<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-002 — Entidade `sabia.inbound_update`

**ID canônico:** `ENT-002` · **ID histórico:** `PST-0102`  
**Banco e schema:** `abioma_sabia` / `sabia` (**propostos**)  
**Estado documental de origem:** `refinement` · **Estado do DDL:** proposta; não há migração executada verificada.

## Objetivo

Registrar cada update externo aceito para fornecer idempotência à entrada Telegram.

## Dependências

- [FLUX-001 (FLW-0001) — Comando Telegram](../../fluxos/FLUX-001-comando-telegram.md)
- [ENT-001 (PST-0101) — transport_cursor](ENT-001-transport-cursor.md)
- [ENT-003 (PST-0103) — request](ENT-003-request.md)

## Estrutura proposta

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `inbound_update_id` | `bigint identity` | não | PostgreSQL | gerado pelo banco | identity |
| `client_id` | `text` | não | configuração | cliente Telegram ativo | copiar |
| `transport` | `text` | não | adaptador | transporte normalizado | `telegram` |
| `transport_update_id` | `bigint` | não | Telegram | `update_id` | inteiro → `bigint` |
| `accepted_at` | `timestamptz` | não | Sabiá | instante do aceite persistente | instante absoluto |
| `raw_update` | `jsonb` | não | Telegram | objeto `Update` integral, sem token | persistir antes do avanço do cursor |
| `processing_status` | `text` | não | Sabiá | `accepted/processing/completed/ignored/failed` | transição controlada |

**UNIQUE proposta:** `(client_id, transport, transport_update_id)`.

## Exemplo ilustrativo

| inbound_update_id | client_id | transport | transport_update_id | accepted_at | raw_update | processing_status |
|---:|---|---|---:|---|---|---|
| 1201 | `bioma` | `telegram` | 48291 | `2026-10-02T23:30:00Z` | `{ "update_id": 48291 }` | `accepted` |

## Origem Telegram

`transport_update_id` é obtido de `Update.update_id`.

## Registro

A tentativa de inserir uma combinação já existente representa update duplicado e não deve gerar nova execução.

## Consumo

É consultada principalmente para:

- verificar idempotência;
- rastrear a origem de uma `request`;
- reconstruir o estado local após restart.

## BLOCKED

**Proposta desta reorganização:** conservar a FK apenas em `request.inbound_update_id`, sem coluna redundante `inbound_update.request_id`. Esta escolha ainda requer aprovação da direção física final.

## Regra de recuperação — decisão de 21/10/2026

A decisão de 21/10/2026 exige o objeto Telegram Update integral em `raw_update` e um ciclo controlado em `processing_status`. O quadro de campos acima incorpora ambos e mantém a unicidade por cliente, transporte e atualização.

Inserir `raw_update` e avançar `transport_cursor.last_update_id` na **mesma transação**. O offset só avança após commit. Reprocessamentos após queda leem `raw_update` localmente. Um claim/lock por update impede processamento concorrente, e resultado existente não deve gerar segunda execução. `raw_update` contém dados pessoais: restringir acesso, excluir dos logs e definir purge seguro depois de todos os trabalhos correlacionados.

## Verificação de consistência

O exemplo acima ilustra apenas a estrutura mínima de `raw_update`; na operação, deve ser persistido o **objeto integral** recebido da API Telegram, sem token. A relação opcional com `request` está em revisão: não definir duas FKs opostas sem decisão explícita. Consulte [request](ENT-003-request.md), [cursor](ENT-001-transport-cursor.md) e [pendências](../PERS-002-pendencias-modelo-fisico.md).

## Chaves, índices e integridade física

- **PK:** `inbound_update_id`; **UNIQUE:** `(client_id, transport, transport_update_id)`.
- **CHECK proposto:** `processing_status IN ('accepted','processing','completed','ignored','failed')`.
- **Índice proposto:** `(processing_status, accepted_at)` para retomada; `raw_update` deve ser acessível apenas ao serviço.

**Estado do DDL:** proposta de tabela; nenhum SQL de migração foi executado.

## Revisão de consistência e pendências

**Recuperação e vínculo:** preservar o `raw_update` integral e o estado de processamento. A direção definitiva da FK para `request` permanece na pendência `DB-03`; não criar duas relações redundantes.

Todas as decisões físicas pendentes permanecem registradas em [pendências PostgreSQL](../PERS-002-pendencias-modelo-fisico.md). A ficha descreve uma **especificação documental**, não comprova instalação do banco nem aprovação do DDL.

## Rastreabilidade

Fonte imediatamente anterior: [PST-0102 — `sabia.inbound_update`](https://github.com/ahhvahh/a-bioma-sabia/blob/d2fdacd8f48253e465d79127d298a1274d6f1269/docs/persistencia/entidades/inbound-update.md). A fonte histórica em `develop`, seus exemplos e os campos originais permanecem rastreáveis no arquivo legado.
