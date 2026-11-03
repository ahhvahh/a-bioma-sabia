# Entidade `sabia.alert_state`

![Tipo](https://img.shields.io/badge/Tipo-entidade-0550ae?style=flat-square) ![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

**Banco proposto:** `abioma_sabia` · **Schema proposto:** `sabia` · **Tabela:** `sabia.alert_state`

**ID:** PST-0107  
**Status:** refinement

## Objetivo

Persistir o estado mínimo necessário para decisões de alerta e lembrete sobreviverem a reinícios.

## Dependências

- [FLW-0003 — Monitoramento agendado](../../especificacao/fluxos/monitoramento-agendado.md)
- [CFG-0001 — Modelo de configuração](../../especificacao/configuracao/modelo-configuracao.md)

## Estrutura e de-para propostos

| Campo | PostgreSQL | Nulo | Origem | Campo de origem | Conversão |
|---|---|---:|---|---|---|
| `schedule_id` | `text` | não | YAML | identificador do agendamento | copiar identificador lógico |
| `state` | `text` | não | Alert Manager | resultado normalizado | `OK/WARNING/CRITICAL/UNKNOWN` |
| `last_evaluated_at` | `timestamptz` | não | Scheduler | instante da avaliação | instante absoluto |
| `last_notified_at` | `timestamptz` | sim | Alert Manager | instante da última notificação | regra exata de atualização ainda BLOCKED |
| `updated_at` | `timestamptz` | não | Sabiá | última alteração | instante absoluto |

### Conversão do resultado de verificação

| Exit code / falha | Estado persistido |
|---|---|
| `0` | `OK` |
| `1` | `WARNING` |
| `2` | `CRITICAL` |
| `3` | `UNKNOWN` |
| falha técnica ou timeout | `UNKNOWN` |

## Exemplo ilustrativo

| schedule_id | state | last_evaluated_at | last_notified_at | updated_at |
|---|---|---|---|---|
| `disk-check` | `WARNING` | `2026-10-02T23:35:00Z` | `2026-10-02T23:35:01Z` | `2026-10-02T23:35:01Z` |

## Consumo

Antes de decidir alerta/lembrete, o Alert Manager lê o estado anterior e `last_notified_at`.

## BLOCKED

- decidir se `last_notified_at` é atualizado ao criar a entrega ou somente após confirmação remota;
- decidir retenção quando um `schedule_id` deixa de existir no YAML.

## Chaves, índices e integridade física

- **PK:** `schedule_id` (identificador no YAML; sem FK física para `schedule`).
- **CHECK proposto:** `state IN ('OK','WARNING','CRITICAL','UNKNOWN')`.
- **Índice proposto:** `(last_evaluated_at)` para relatórios de monitoramento. O momento de atualizar `last_notified_at` está em aberto.

**Estado do DDL:** proposta em revisão; nenhuma tabela foi criada.
