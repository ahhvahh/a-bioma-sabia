# Jobs

![MOD](https://img.shields.io/badge/MOD-MOD--0004-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Executar operações demoradas sem bloquear o recebimento de comandos e garantir que a resposta seja entregue ao cliente correto.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0006 — Jobs assíncronos](../../adr/processamento/jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)
- [CTR-0003 — Job](../contratos/job.md)

## Responsabilidades

- criar e persistir job;
- enfileirar trabalho;
- limitar concorrência por configuração;
- executar por workers;
- controlar estados e histórico;
- publicar progresso;
- correlacionar resultado com `client_id` e destino de resposta;
- persistir respostas até a confirmação de entrega;
- recuperar fila após reinício;
- suportar consulta e cancelamento conforme CTR-0003.

## Entradas

Solicitação de operação assíncrona e contexto persistente de resposta.

## Saídas

Identificador de job, mudanças de estado, progresso e resposta final destinada ao cliente correspondente.

## Interfaces e contratos

- [CTR-0003 — Job](../contratos/job.md)

## Persistência

SQLite é o armazenamento oficial. Jobs `queued` sobrevivem ao reinício; jobs encontrados em `running` após reinício passam para `failed/service_restart`. Respostas não entregues permanecem pendentes.

## Restrições

- estados e transições seguem CTR-0003;
- não existe retry automático por padrão;
- limites `max_workers` e `max_pending` vêm da configuração.

## Critérios de aceite

- job não bloqueia o recebimento de comandos;
- fila e concorrência respeitam configuração;
- resposta é entregue usando o cliente e destino persistidos;
- queued sobrevive a restart;
- running interrompido não é reexecutado silenciosamente;
- cancelamento e retry seguem CTR-0003.

## Implementação relacionada

Prevista para os pacotes internos de jobs, fila e workers.
