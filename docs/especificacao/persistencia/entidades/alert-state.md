# Entidade `alert_state`

**ID:** PST-0107  
**Status:** refinement

## Objetivo

Persistir o estado mínimo necessário para decisões de alerta e lembrete sobreviverem a reinícios.

## Dependências

- [FLW-0003 — Monitoramento agendado](../../fluxos/monitoramento-agendado.md)
- [CFG-0001 — Modelo de configuração](../../configuracao/modelo-configuracao.md)

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
