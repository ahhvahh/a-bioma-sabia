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
3. Script Executor executa com timeout;
4. resultado é convertido em `OK`, `WARNING`, `CRITICAL` ou `UNKNOWN`;
5. Alert Manager recupera o estado anterior persistido;
6. compara estado atual e anterior;
7. se houver transição relevante, cria mensagem;
8. adaptador do cliente Alerts envia ao Telegram;
9. novo estado é persistido no SQLite para avaliação futura.

## Fluxos alternativos

### Estado inalterado

Não enviar imediatamente; lembrete só quando configurado.

### Recuperação

Transição de estado degradado para `OK` envia mensagem de normalização.

### Tarefa sempre-reportar

Pode enviar resultado mesmo sem mudança quando essa opção estiver configurada.

## Falhas e tratamento

O estado anterior sobrevive ao reinício por meio do SQLite. A política detalhada de falha do scheduler e de lembretes ainda precisa ser fechada.

## Resultado

Estado atualizado e persistido e, quando necessário, alerta enviado.

## Critérios de aceite

Primeira prova funcional: `disk-check` com limites configuráveis:
- abaixo de 80%: `OK`;
- 80% a 89%: `WARNING`;
- 90% ou mais: `CRITICAL`.

Os valores são exemplos iniciais e devem ser configuráveis.

## Implementação relacionada

Scheduler, Script Executor, Alert Manager, SQLite e cliente Alerts.
