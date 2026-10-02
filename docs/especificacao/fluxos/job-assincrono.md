# Job assíncrono

![FLW](https://img.shields.io/badge/FLW-FLW--0002-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Executar uma operação demorada sem bloquear o tratamento de novos comandos e manter o usuário informado.

## Dependências

- [MOD-0004 — Jobs](../modulos/jobs.md)
- [CTR-0003 — Job](../contratos/job.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)

## Gatilho

Command Router identifica uma operação classificada como demorada.

## Pré-condições

- operação válida e autorizada;
- contexto de resposta disponível.

## Fluxo principal

1. Job Manager cria e persiste a requisição/job;
2. o job entra em `queued`;
3. o usuário recebe confirmação imediata com identificador do job;
4. Job Queue disponibiliza o trabalho;
5. Worker inicia e muda para `running`;
6. Worker publica progresso quando aplicável;
7. adaptador Telegram edita a mensagem associada;
8. Worker conclui como `completed`, `failed`, `cancelled` ou `timeout`;
9. o resultado a entregar permanece persistido enquanto estiver pendente;
10. resultado final é enviado; se houver arquivo, ele é transmitido ao usuário;
11. a entrega é registrada no estado operacional.

## Fluxos alternativos

### Reinício com mensagem pendente

1. o serviço lê o SQLite no startup;
2. identifica respostas prontas ainda não entregues;
3. devolve essas respostas para a etapa de envio ao destino correlacionado.

### Consulta de job

`/jobs` lista jobs visíveis ao cliente/usuário conforme autorização e `/job <id>` consulta um job.

### Cancelamento

Previsto para o cliente Tools, mas a semântica ainda não está definida.

## Falhas e tratamento

A persistência de requisições e respostas é definida por ADR-0009. Retry e comportamento de job que estava executando durante interrupção dependem de CTR-0003 e ADR-0010.

## Resultado

Job termina em estado final observável e a resposta pendente permanece recuperável até sua entrega.

## Critérios de aceite

- recebimento de comandos continua enquanto job executa;
- progresso pode editar a mesma mensagem;
- resposta pendente sobrevive ao reinício;
- política de concorrência, retry e job interrompido precisa ser definida antes de `refined`.

## Implementação relacionada

JobManager, JobQueue, Worker, SQLite e adaptador Telegram.
