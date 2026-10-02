# Encerramento do serviço

![FLW](https://img.shields.io/badge/FLW-FLW--0004-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Encerrar o Sabiá de forma controlada ao receber sinal do sistema operacional.

## Dependências

- [ADR-0001 — Go, binário único e serviço Linux](../../adr/runtime/go-binario-unico.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [MOD-0004 — Jobs](../modulos/jobs.md)
- [MOD-0005 — Scheduler e Alert Manager](../modulos/scheduler-alertas.md)

## Gatilho

Recebimento de `SIGTERM` ou `SIGINT`.

## Pré-condições

Serviço em execução.

## Fluxo principal

1. parar de aceitar novos jobs;
2. interromper criação de novas execuções agendadas;
3. tratar jobs em execução conforme política;
4. finalizar workers;
5. salvar o estado definido como persistente;
6. fechar recursos;
7. encerrar o processo.

## Fluxos alternativos

### Job em execução

**BLOCKED:** comportamento depende de ADR-0010.

## Falhas e tratamento

Tempo máximo de shutdown e comportamento quando uma operação não encerra ainda não foram definidos.

## Resultado

Processo encerrado sem aceitar novo trabalho após início do shutdown.

## Critérios de aceite

Para `refined`, fechar ADR-0009 e ADR-0010 e definir timeout de encerramento se aplicável.

## Implementação relacionada

Processo principal, Job Manager, Scheduler e recursos compartilhados.
