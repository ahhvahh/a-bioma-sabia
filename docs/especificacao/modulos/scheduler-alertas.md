# Scheduler e Alert Manager

![MOD](https://img.shields.io/badge/MOD-MOD--0005-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Executar verificações periódicas e notificar clientes de alerta apenas quando houver evento relevante.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0007 — Scheduler e alertas orientados a estado](../../adr/monitoramento/scheduler-alertas-estado.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [CTR-0002 — Execução de script](../contratos/execucao-script.md)

## Responsabilidades

- disparar tarefas cadastradas por periodicidade;
- interpretar status de monitoramento;
- comparar estado atual e anterior;
- persistir estado necessário à avaliação futura;
- emitir alerta em mudança relevante;
- emitir recuperação;
- suprimir repetição imediata;
- permitir lembrete periódico configurado;
- impedir sobreposição da mesma tarefa agendada: se um novo disparo ocorrer enquanto a execução anterior da mesma tarefa ainda estiver ativa, o novo disparo é ignorado e o evento é registrado.

## Entradas

Agendamentos e resultados de verificações.

## Saídas

Nenhuma mensagem quando não houver evento relevante, salvo tarefa configurada para sempre reportar; mensagem de alerta/recuperação quando aplicável.

## Interfaces e contratos

- [CTR-0002 — Execução de script](../contratos/execucao-script.md)

## Persistência

O estado anterior necessário à avaliação de alertas deve ser mantido no SQLite conforme ADR-0009 e recuperado após reinício.

## Restrições

Estados de monitoramento: `OK`, `WARNING`, `CRITICAL`, `UNKNOWN`.

Para uma mesma tarefa agendada, somente uma execução pode permanecer ativa. Se a periodicidade vencer novamente antes do término da execução corrente, o Scheduler não cria nova execução, não enfileira uma segunda ocorrência e registra o disparo ignorado para observabilidade.

## Critérios de aceite

- transição gera alerta conforme regra;
- repetição de estado não gera mensagem imediata;
- recuperação gera mensagem;
- estado anterior pode ser recuperado após reinício;
- uma mesma tarefa não possui execuções sobrepostas;
- disparo ocorrido durante execução ativa é ignorado e registrado;
- política detalhada de lembrete e comportamento de falhas ainda precisa ser completada antes de `refined`.

## Implementação relacionada

Prevista para os pacotes internos de scheduler e alerts.
