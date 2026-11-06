<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ENT-007 — Entidade `sabia.alert_state`

**ID canônico:** `ENT-007` · **ID histórico:** `PST-0107`  
**Banco e schema:** `abioma_sabia` / `sabia` (**propostos**)  
**Estado documental de origem:** `refinement` · **Estado do DDL:** proposta; não há migração executada verificada.

## Objetivo

Persistir o estado mínimo necessário para decisões de alerta e lembrete sobreviverem a reinícios.

## Dependências

- [FLUX-003 (FLW-0003) — Monitoramento agendado](../../fluxos/FLUX-003-monitoramento-agendado.md)
- [REF-001 (CFG-0001) — Modelo de configuração](../../referencia/REF-001-modelo-configuracao.md)

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

## Verificação de consistência

O agendamento é identificado por `schedule_id` no YAML, **sem tabela `schedule` ou FK correspondente**. O estado `UNKNOWN` inclui falhas técnicas e timeout. O instante de atualizar `last_notified_at` e a retenção de agendamentos removidos continuam em aberto. Consulte [fluxo do Scheduler](../../fluxos/FLUX-003-monitoramento-agendado.md) e [pendências](../PERS-002-pendencias-modelo-fisico.md).

## Chaves, índices e integridade física

- **PK:** `schedule_id` (identificador no YAML; sem FK física para `schedule`).
- **CHECK proposto:** `state IN ('OK','WARNING','CRITICAL','UNKNOWN')`.
- **Índice proposto:** `(last_evaluated_at)` para relatórios de monitoramento. O momento de atualizar `last_notified_at` está em aberto.

**Estado do DDL:** proposta em revisão; nenhuma tabela foi criada.

## Revisão de consistência e pendências

**Configuração do agendamento:** `schedule_id` vem do YAML, sem FK para tabela de schedules. `last_notified_at`, retries e retenção ainda exigem definição.

Todas as decisões físicas pendentes permanecem registradas em [pendências PostgreSQL](../PERS-002-pendencias-modelo-fisico.md). A ficha descreve uma **especificação documental**, não comprova instalação do banco nem aprovação do DDL.

## Rastreabilidade

Fonte imediatamente anterior: [PST-0107 — `sabia.alert_state`](alert-state.md). A fonte histórica em `develop`, seus exemplos e os campos originais permanecem rastreáveis no arquivo legado.
