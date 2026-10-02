# Jobs

![MOD](https://img.shields.io/badge/MOD-MOD--0004-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Executar operações demoradas sem bloquear o recebimento de comandos e garantir que a resposta seja entregue ao cliente correto.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0006 — Jobs assíncronos](../../adr/processamento/jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)
- [CTR-0003 — Job](../contratos/job.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)
- [MOD-0007 — Processadores assíncronos e transporte](processadores-assincronos.md)

## Responsabilidades

- criar e persistir job;
- enfileirar trabalho;
- limitar concorrência por configuração;
- executar por workers;
- controlar estados e histórico;
- receber e publicar progresso `loading` correlacionado por `request_id`;
- receber eventos `content` correlacionados e encaminhar cada arquivo ao pipeline de entrega;
- aceitar `finally` como finalização semântica normal do processador;
- correlacionar resultado com `client_id` e destino de resposta;
- persistir referência/estado de cada conteúdo anunciado e a resposta final antes da entrega;
- persistir respostas até a confirmação de entrega;
- recuperar fila após reinício;
- suportar consulta e cancelamento conforme CTR-0003.

## Entradas

Solicitação de operação assíncrona e contexto persistente de resposta.

## Saídas

Identificador de job, mudanças de estado, eventos `loading`, conteúdos `content` e resposta `finally` destinada ao cliente correspondente.

## Interfaces e contratos

- [CTR-0003 — Job](../contratos/job.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)

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
- cancelamento e retry seguem CTR-0003;
- `loading` não finaliza o job;
- `content` pode ocorrer várias vezes e não finaliza o job;
- `finally` correlacionado conclui semanticamente o processamento;
- permanece `refinement` enquanto CTR-0005 estiver incompleto para framing e mídia.

## Implementação relacionada

Prevista para os pacotes internos de jobs, fila e workers.
