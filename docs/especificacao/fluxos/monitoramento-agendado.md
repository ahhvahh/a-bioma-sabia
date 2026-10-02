# Monitoramento agendado

![FLW](https://img.shields.io/badge/FLW-FLW--0003-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Executar verificações periódicas e enviar alertas apenas quando a avaliação produzir evento relevante.

## Dependências

- [MOD-0005 — Scheduler e Alert Manager](../modulos/scheduler-alertas.md)
- [CTR-0002 — Execução de script](../contratos/execucao-script.md)
- [ADR-0007 — Scheduler e alertas orientados a estado](../../adr/monitoramento/scheduler-alertas-estado.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)

## Gatilho

Agendamento configurado torna-se devido.

## Pré-condições

- tarefa habilitada;
- script cadastrado;
- cliente de alerta habilitado quando houver notificação.

## Fluxo principal

1. Scheduler identifica tarefa devida;
2. resolve script cadastrado;
3. Script Executor executa conforme CTR-0002;
4. Scheduler/Alert Manager interpreta o exit code de monitoramento;
5. resultado é convertido em `OK`, `WARNING`, `CRITICAL` ou `UNKNOWN`;
6. Alert Manager recupera o estado anterior persistido;
7. compara estado atual e anterior;
8. se houver transição relevante, cria mensagem;
9. adaptador do cliente Alerts envia ao Telegram;
10. novo estado é persistido no SQLite para avaliação futura.

Para scripts de monitoramento, a interpretação inicial é:

- `0 = OK`;
- `1 = WARNING`;
- `2 = CRITICAL`;
- `3 = UNKNOWN`.

O Script Executor não atribui significado de monitoramento ao exit code.

## Fluxos alternativos

### Novo disparo durante execução ativa

1. Scheduler identifica que a mesma tarefa já possui uma execução ativa;
2. o novo disparo é ignorado;
3. nenhuma nova execução é criada ou enfileirada;
4. o disparo ignorado é registrado para observabilidade;
5. a execução já ativa continua normalmente.

### Primeira avaliação sem estado anterior

1. Alert Manager identifica ausência de estado anterior persistido;
2. se o estado atual for `OK`, persiste `OK` sem enviar alerta;
3. se o estado atual for `WARNING`, `CRITICAL` ou `UNKNOWN`, cria e envia alerta imediatamente;
4. o estado atual é persistido como referência para as próximas avaliações.

### Estado inalterado

Não enviar imediatamente; lembrete só quando configurado.

### Recuperação

Transição de estado degradado para `OK` envia mensagem de normalização.

### Tarefa sempre-reportar

Pode enviar resultado mesmo sem mudança quando essa opção estiver configurada.

## Falhas e tratamento

O estado anterior sobrevive ao reinício por meio do SQLite. A política detalhada de falha do scheduler e de lembretes ainda não está fechada.

## Resultado

Estado atualizado e persistido e, quando necessário, alerta enviado.

## Critérios de aceite

Primeira prova funcional: `disk-check` com limites configuráveis:

- abaixo de 80%: `OK`;
- 80% a 89%: `WARNING`;
- 90% ou mais: `CRITICAL`.

Os valores são exemplos iniciais e devem ser configuráveis.

- primeira avaliação em `OK` não gera alerta;
- primeira avaliação em `WARNING`, `CRITICAL` ou `UNKNOWN` gera alerta;
- uma tarefa que vença novamente enquanto sua execução anterior estiver ativa não inicia execução concorrente;
- o disparo sobreposto é ignorado e registrado.

O fluxo permanece em `refinement` até fechar as demais pendências específicas de Scheduler e Alertas.

## Implementação relacionada

Scheduler, Script Executor, Alert Manager, SQLite e cliente Alerts.
