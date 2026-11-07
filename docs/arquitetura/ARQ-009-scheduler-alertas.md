<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ARQ-009 — Scheduler e Alert Manager

**ID:** `ARQ-009` · **Origem histórica:** `MOD-0005`  
**Estado original:** `refinement` · **Implementação:** não verificada nesta branch  
**Classificação:** revisão documental; itens em `refinement` não foram aprovados por esta migração.

## Objetivo

Executar verificações periódicas e notificar clientes de alerta apenas quando houver evento relevante.

## Dependências

- [ARQ-002 (DSG-0002) — Componentes do Sabiá Core](ARQ-002-componentes-core.md)
- [DEC-007 (ADR-0007) — Scheduler e alertas orientados a estado](../decisoes/DEC-007-alertas-estado.md)
- [DEC-009 (ADR-0009) — Persistência do estado operacional](../decisoes/DEC-009-postgresql-estado-operacional.md)
- [CONTR-002 (CTR-0002) — Execução de script](../contratos/CONTR-002-execucao-script.md)

## Responsabilidades

- disparar tarefas cadastradas por `interval` de duração;
- interpretar status de monitoramento;
- converter falha técnica da verificação ou timeout em estado `UNKNOWN`, preservando o motivo para observabilidade;
- comparar estado atual e anterior;
- persistir estado necessário à avaliação futura;
- na primeira avaliação sem estado anterior, persistir silenciosamente `OK` e notificar imediatamente `WARNING`, `CRITICAL` ou `UNKNOWN`;
- emitir alerta em mudança relevante;
- emitir recuperação;
- suprimir repetição imediata;
- permitir lembrete periódico configurado por `reminder_interval` opcional em cada agendamento;
- impedir sobreposição da mesma tarefa agendada: se um novo disparo ocorrer enquanto a execução anterior da mesma tarefa ainda estiver ativa, o novo disparo é ignorado e o evento é registrado.

## Entradas

Agendamentos e resultados de verificações.

## Saídas

Nenhuma mensagem quando não houver evento relevante, salvo tarefa configurada para sempre reportar; mensagem de alerta/recuperação quando aplicável.

## Interfaces e contratos

- [CONTR-002 (CTR-0002) — Execução de script](../contratos/CONTR-002-execucao-script.md)

## Persistência

O estado anterior necessário à avaliação de alertas deve ser mantido no PostgreSQL conforme ADR-0009 e recuperado após reinício.

## Restrições

Estados de monitoramento: `OK`, `WARNING`, `CRITICAL`, `UNKNOWN`.

No MVP, cada agendamento utiliza exclusivamente um `interval` de duração positivo. Expressões cron, calendários e horários absolutos não fazem parte do primeiro MVP.

Para uma mesma tarefa agendada, somente uma execução pode permanecer ativa. Se a periodicidade vencer novamente antes do término da execução corrente, o Scheduler não cria nova execução, não enfileira uma segunda ocorrência e registra o disparo ignorado para observabilidade.

Falha técnica ao iniciar/executar a verificação, erro interno do executor ou timeout da verificação produz estado `UNKNOWN`. O motivo técnico original deve permanecer disponível em logs/auditoria. O estado `UNKNOWN` participa normalmente das regras de primeira avaliação, transição, repetição e recuperação.

Quando o estado permanecer em `WARNING`, `CRITICAL` ou `UNKNOWN`, um lembrete só pode ser enviado se o agendamento possuir `reminder_interval` configurado e o intervalo tiver transcorrido desde a última notificação desse estado. Sem `reminder_interval`, não existem lembretes periódicos. Qualquer mudança de estado reinicia a contagem e a recuperação para `OK` encerra os lembretes.

## Critérios de aceite

- transição gera alerta conforme regra;
- repetição de estado não gera mensagem imediata;
- recuperação gera mensagem;
- estado anterior pode ser recuperado após reinício;
- primeira avaliação em `OK` é persistida sem alerta;
- primeira avaliação em `WARNING`, `CRITICAL` ou `UNKNOWN` gera alerta e persiste o estado;
- uma mesma tarefa não possui execuções sobrepostas;
- disparo ocorrido durante execução ativa é ignorado e registrado;
- falha técnica ou timeout da verificação resulta em `UNKNOWN`;
- transição para `UNKNOWN` é persistida e pode gerar alerta conforme as mesmas regras dos demais estados;
- estado degradado inalterado só gera lembrete quando `reminder_interval` estiver configurado e vencido;
- sem `reminder_interval`, estado inalterado não gera lembretes;
- mudança de estado reinicia a contagem de lembrete e `OK` encerra lembretes;
- todo agendamento do MVP possui `interval` positivo; cron não é aceito.

## Implementação relacionada

Prevista para os pacotes internos de scheduler e alerts.

## Pendências e integração

A correlação de alertas do Scheduler em `outbound_message` sem `request_id`, o registro de `last_notified_at` e o schema final dos agendamentos continuam pendentes. Consultar [fluxo de monitoramento](../fluxos/FLUX-003-monitoramento-agendado.md) e [alert_state](../persistencia/entidades/ENT-007-alert-state.md).

## Rastreabilidade

Documento original preservado: [MOD-0005 — Scheduler e Alert Manager](../especificacao/modulos/scheduler-alertas.md).
