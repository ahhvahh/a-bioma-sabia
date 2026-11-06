<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-004 — Entidade `sabia.job`

**ID canônico:** `ENT-004` · **ID histórico:** `PST-0104`  
**Banco e schema:** `abioma_sabia` / `sabia` (**propostos**)  
**Estado documental de origem:** `refined` · **Estado do DDL:** proposta; não há migração executada verificada.

## Objetivo

Persistir o estado atual de uma operação assíncrona conforme CTR-0003.

## Dependências

- [CONTR-003 (CTR-0003) — Job](../../contratos/CONTR-003-job.md)
- [ENT-003 (PST-0103) — request](ENT-003-request.md)
- [ENT-005 (PST-0105) — job_state_history](ENT-005-job-state-history.md)
- [PST-0003 — Claim concorrente de filas persistentes](../concorrencia.md)

## Estrutura e de-para propostos

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `job_id` | `bigint identity` | não | PostgreSQL | gerado pelo banco | identity |
| `request_id` | `uuid` | não | `request` | `request.request_id` | copiar |
| `client_id` | `text` | não | `request` | `request.client_id` | copiar |
| `principal_id` | `text` | não | `request` | `request.principal_id` | copiar |
| `transport` | `text` | não | `request` | `request.transport` | copiar |
| `destination_id` | `text` | não | `request` | `request.destination_id` | copiar |
| `operation` | `text` | não | Command Router | operação resolvida | identificador lógico da operação |
| `status` | `text` | não | Job Manager | estado CTR-0003 | domínio controlado |
| `created_at` | `timestamptz` | não | Job Manager | criação | instante absoluto |
| `started_at` | `timestamptz` | sim | worker | transição para `running` | instante absoluto |
| `finished_at` | `timestamptz` | sim | worker/Job Manager | transição terminal | instante absoluto |
| `updated_at` | `timestamptz` | não | Sabiá | última transição | instante absoluto |
| `terminal_reason` | `text` | sim | Job Manager/runtime | motivo terminal | código como `service_restart` quando aplicável |

## Exemplo ilustrativo

| job_id | request_id | client_id | operation | status | created_at | started_at | finished_at | terminal_reason |
|---:|---|---|---|---|---|---|---|---|
| 81 | `550e8400-e29b-41d4-a716-446655440000` | `bioma` | `system.status` | `completed` | `2026-10-02T23:30:02Z` | `2026-10-02T23:30:03Z` | `2026-10-02T23:30:04Z` | `NULL` |

## Registro

A criação do job e o primeiro `job_state_history` devem ocorrer na mesma transação.

## Consumo

Workers não consomem `queued` por leitura simples. O próximo job é obtido pelo claim atômico de PST-0003, que realiza `queued → running` e registra o histórico correspondente na mesma transação antes de devolver o job ao worker. Recovery continua consultando `queued` e `running` segundo CTR-0003.

## Cardinalidade

Uma mesma `request_id` pode possuir zero ou vários jobs. `job.request_id` não possui restrição `UNIQUE`.


## Integridade referencial

`job.request_id → request.request_id` usa `ON DELETE RESTRICT`.

O job deve ser removido explicitamente pelo processo de limpeza antes da remoção da `request`. Não existe `CASCADE` nem `SET NULL` no MVP.

## Verificação de consistência

Um `request_id` pode originar **mais de um job**; por isso, não criar `UNIQUE(job.request_id)`. A criação inicial e as transições de estado atualizam [histórico](ENT-005-job-state-history.md) na mesma transação. A reserva concorrente segue [PST-0003](../concorrencia.md).

## Chaves, índices e integridade física

- **PK:** `job_id`; **FK:** `request_id → request.request_id ON DELETE RESTRICT` (N jobs por request).
- **CHECK proposto:** `status IN ('queued','running','completed','failed','cancelled','timeout')`.
- **Índices propostos:** `(status, created_at, job_id)` para claim; `(request_id)` para correlação.

**Estado do DDL:** proposta de tabela; nenhum SQL de migração foi executado.

## Revisão de consistência e pendências

**Cardinalidade:** uma `request` pode originar vários jobs. O claim e a inclusão de histórico inicial são requisitos documentados; workers não foram verificados nesta branch.

Todas as decisões físicas pendentes permanecem registradas em [pendências PostgreSQL](../pendencias.md). A ficha descreve uma **especificação documental**, não comprova instalação do banco nem aprovação do DDL.

## Rastreabilidade

Fonte imediatamente anterior: [PST-0104 — `sabia.job`](job.md). A fonte histórica em `develop`, seus exemplos e os campos originais permanecem rastreáveis no arquivo legado.
