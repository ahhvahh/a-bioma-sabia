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

1. Job Manager cria o job em `queued`;
2. o usuário recebe confirmação imediata com identificador do job;
3. Job Queue disponibiliza o trabalho;
4. Worker inicia e muda para `running`;
5. Worker publica progresso quando aplicável;
6. adaptador Telegram edita a mensagem associada;
7. Worker conclui como `completed`, `failed`, `cancelled` ou `timeout`;
8. resultado final é enviado; se houver arquivo, ele é transmitido ao usuário.

## Fluxos alternativos

### Consulta de job

`/jobs` lista jobs visíveis ao cliente/usuário conforme autorização e `/job <id>` consulta um job.

### Cancelamento

Previsto para o cliente Tools, mas a semântica ainda não está definida.

## Falhas e tratamento

Recovery, retries e interrupção por shutdown dependem de ADR-0009 e ADR-0010.

## Resultado

Job termina em estado final observável.

## Critérios de aceite

- recebimento de comandos continua enquanto job executa;
- progresso pode editar a mesma mensagem;
- política de concorrência/restart precisa ser definida antes de `refined`.

## Implementação relacionada

JobManager, JobQueue, Worker e adaptador Telegram.
