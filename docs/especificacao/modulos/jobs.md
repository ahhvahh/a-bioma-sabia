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
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [MOD-0007 — Processadores assíncronos e transporte](processadores-assincronos.md)

## Responsabilidades

- criar e persistir job;
- enfileirar trabalho;
- limitar concorrência pela quantidade de threads lógicas de CPU disponíveis ao processo no startup;
- executar por workers;
- controlar estados e histórico;
- receber e publicar progresso `loading` correlacionado por `request_id`;
- aceitar `finally` como finalização semântica normal do processador;
- correlacionar mídia persistida pelo socket de ingestão usando `request_id`;
- correlacionar resultado com `client_id` e destino de resposta;
- acompanhar transmissões CTR-0007 sem transportar binário pelo job;
- persistir respostas até a confirmação de entrega;
- recuperar fila após reinício;
- suportar consulta e cancelamento conforme CTR-0003.

## Entradas

Solicitação de operação assíncrona e contexto persistente de resposta.

## Saídas

Identificador de job, mudanças de estado, eventos `loading`, resposta `finally` e correlação com mídias/transmissões associadas ao mesmo `request_id`.

## Interfaces e contratos

- [CTR-0003 — Job](../contratos/job.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)

## Persistência

PostgreSQL é o armazenamento oficial. Jobs `queued` sobrevivem ao reinício; jobs encontrados em `running` após reinício passam para `failed/service_restart`. Respostas não entregues permanecem pendentes. Transmissões de mídia `pending` ou `transmitting` são retomadas conforme CTR-0007 quando o `media_id` existe.

## Restrições

- estados e transições seguem CTR-0003;
- não existe retry automático por padrão;
- o limite de execução simultânea é derivado das threads lógicas de CPU disponíveis ao processo;
- apenas `max_pending` vem da configuração para limitar a fila `queued`;
- uma mesma `request_id` pode correlacionar zero ou vários jobs.

## Critérios de aceite

- job não bloqueia o recebimento de comandos;
- fila respeita `max_pending` e concorrência respeita as threads lógicas de CPU disponíveis ao processo;
- resposta é entregue usando o cliente e destino persistidos;
- queued sobrevive a restart;
- running interrompido não é reexecutado silenciosamente;
- cancelamento e retry seguem CTR-0003;
- `loading` não finaliza o job;
- uploads de mídia são independentes do canal de controle e usam o mesmo `request_id`;
- `finally` correlacionado conclui semanticamente o processamento;
- transmissão de mídia ativa permanece recuperável após restart enquanto o `media_id` existir;
- CTR-0005 está `refined`; o módulo usa scripts Bash registrados e concorrência derivada das threads lógicas disponíveis.

## Implementação relacionada

Prevista para os pacotes internos de jobs, fila e workers.
