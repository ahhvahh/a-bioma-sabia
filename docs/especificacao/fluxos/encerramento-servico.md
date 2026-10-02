# Encerramento do serviço

![FLW](https://img.shields.io/badge/FLW-FLW--0004-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Encerrar o Sabiá de forma controlada, avisando todos os clientes Telegram ativos e preservando a entrega de respostas.

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

1. entrar em estado de shutdown;
2. parar de aceitar novas requisições/jobs;
3. impedir novos disparos do Scheduler;
4. avisar todos os clientes Telegram ativos por seus destinos autorizados conhecidos;
5. manter os canais de saída necessários para concluir respostas já pendentes;
6. aguardar jobs em `running` até `service.shutdown_timeout`;
7. enviar toda resposta concluída ao cliente e destino que originaram a requisição;
8. manter no SQLite respostas que ainda não puderem ser entregues;
9. preservar jobs `queued` para o próximo startup;
10. ao atingir o timeout, cancelar jobs ainda executando e registrá-los como `failed/shutdown_timeout`;
11. finalizar workers;
12. persistir estado final;
13. fechar recursos;
14. encerrar.

## Fluxos alternativos

### Falha ao avisar um cliente

Registrar a falha e continuar o shutdown. O aviso não pode impedir indefinidamente o encerramento.

### Resposta pronta sem entrega

Persistir como pendente para reenvio no próximo startup.

## Falhas e tratamento

O tempo máximo é definido por `service.shutdown_timeout`. O encerramento não aguarda indefinidamente.

## Resultado

O serviço encerra sem aceitar novo trabalho, com fila e respostas pendentes preservadas.

## Critérios de aceite

- todos os clientes ativos recebem tentativa de aviso;
- respostas concluídas durante shutdown são direcionadas ao cliente correto;
- fila pendente sobrevive ao restart;
- execução que excede timeout fica registrada como falha;
- respostas não entregues permanecem recuperáveis.

## Implementação relacionada

Processo principal, adaptador Telegram, Job Manager, Scheduler, workers e SQLite.
