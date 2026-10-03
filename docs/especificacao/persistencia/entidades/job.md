# Entidade `job`

**ID:** PST-0104  
**Status:** refinement

## Objetivo

Persistir o estado atual de uma operação assíncrona conforme CTR-0003.

## Dependências

- [CTR-0003 — Job](../../contratos/job.md)
- [PST-0103 — request](request.md)
- [PST-0105 — job_state_history](job-state-history.md)
- [PST-0003 — Claim concorrente de filas persistentes](../claim-concorrente-filas.md)

## Estrutura e de-para propostos

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `job_id` | `bigint identity` | não | PostgreSQL | gerado pelo banco | identity |
| `request_id` | `text` | não | `request` | `request.request_id` | copiar |
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
| 81 | `req-example-001` | `bioma` | `system.status` | `completed` | `2026-10-02T23:30:02Z` | `2026-10-02T23:30:03Z` | `2026-10-02T23:30:04Z` | `NULL` |

## Registro

A criação do job e o primeiro `job_state_history` devem ocorrer na mesma transação.

## Consumo

Workers não consomem `queued` por leitura simples. O próximo job é obtido pelo claim atômico de PST-0003, que realiza `queued → running` e registra o histórico correspondente na mesma transação antes de devolver o job ao worker. Recovery continua consultando `queued` e `running` segundo CTR-0003.

## BLOCKED

Confirmar se uma mesma request pode criar mais de um job. O modelo atual permite `1:N`.
