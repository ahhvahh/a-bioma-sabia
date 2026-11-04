<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# Entidade `sabia.job_state_history`

![Tipo](https://img.shields.io/badge/Tipo-entidade-0550ae?style=flat-square) ![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

**Banco proposto:** `abioma_sabia` · **Schema proposto:** `sabia` · **Tabela:** `sabia.job_state_history`

**ID:** PST-0105  
**Status:** refinement

## Objetivo

Preservar o histórico imutável das mudanças de estado de cada job.

## Dependências

- [CTR-0003 — Job](../../especificacao/contratos/job.md)
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

## Integridade referencial

`job_state_history.job_id → job.job_id` usa `ON DELETE RESTRICT`.

O histórico elegível deve ser removido explicitamente antes do job correspondente. Não existe exclusão implícita em cascata.

## BLOCKED

A política de retenção histórica ainda não está definida.

## Verificação de consistência

O estado corrente pertence a [`job`](job.md); esta tabela conserva eventos de transição, sem substituir o registro anterior. A aplicação deve criar o estado inicial na mesma transação de criação do job. A política de retenção permanece sem decisão, como documentado em [pendências](../pendencias.md).

## Chaves, índices e integridade física

- **PK:** `job_state_history_id`; **FK:** `job_id → job.job_id ON DELETE RESTRICT`.
- **Índice proposto:** `(job_id, changed_at, job_state_history_id)` para reproduzir transições.
- **Imutabilidade:** histórico é somente inserido; alteração de status e inclusão do histórico são uma transação.

**Estado do DDL:** proposta de tabela; nenhum SQL de migração foi executado.

**Situação documental:** ficha revisada no A-Bioma Doc Standard; `refinement` é o estado de refinamento original, não prova de implementação. **Origem histórica:** [PST correspondente em `develop`](https://github.com/ahhvahh/a-bioma-sabia/blob/96427358088a6663176f0fb96de6b14166f975e4/docs/especificacao/persistencia/entidades/job-state-history.md).
