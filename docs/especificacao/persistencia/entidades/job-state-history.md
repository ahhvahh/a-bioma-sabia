# Entidade `job_state_history`

**ID:** PST-0105  
**Status:** refinement

## Objetivo

Preservar o histórico imutável das mudanças de estado de cada job.

## Dependências

- [CTR-0003 — Job](../../contratos/job.md)
- [PST-0104 — job](job.md)

## Estrutura e de-para propostos

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `job_state_history_id` | `bigint identity` | não | PostgreSQL | gerado | identity |
| `job_id` | `bigint` | não | `job` | `job.job_id` | copiar |
| `status` | `text` | não | Job Manager | novo estado CTR-0003 | copiar valor do domínio |
| `reason` | `text` | sim | Job Manager/runtime | motivo da transição | código controlado quando existir |
| `changed_at` | `timestamptz` | não | Sabiá | instante da transição | instante absoluto |

## Exemplo ilustrativo

| job_state_history_id | job_id | status | reason | changed_at |
|---:|---:|---|---|---|
| 1001 | 81 | `queued` | `NULL` | `2026-10-02T23:30:02Z` |
| 1002 | 81 | `running` | `NULL` | `2026-10-02T23:30:03Z` |
| 1003 | 81 | `completed` | `NULL` | `2026-10-02T23:30:04Z` |

## Registro

Nunca atualizar um histórico existente para representar outro estado. Cada transição cria nova linha.

## Consumo

Usado para rastreabilidade e diagnóstico da evolução do job; o estado corrente permanece em `job.status`.

## BLOCKED

A política de retenção histórica ainda não está definida.
