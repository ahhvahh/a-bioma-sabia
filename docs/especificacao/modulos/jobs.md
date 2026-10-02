# Jobs

![MOD](https://img.shields.io/badge/MOD-MOD--0004-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Executar operações demoradas sem bloquear o recebimento de comandos.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0006 — Jobs assíncronos](../../adr/processamento/jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)
- [CTR-0003 — Job](../contratos/job.md)

## Responsabilidades

- criar job e responder imediatamente;
- enfileirar trabalho;
- executar por workers;
- controlar estados válidos;
- publicar progresso;
- correlacionar resultado com destino de resposta;
- suportar consulta por `/jobs` e `/job <id>`;
- suportar cancelamento quando previsto pelo cliente.

## Entradas

Solicitação de operação assíncrona e contexto necessário para resposta.

## Saídas

Identificador de job, mudanças de estado, progresso e resultado final.

## Interfaces e contratos

- [CTR-0003 — Job](../contratos/job.md)

## Persistência

**BLOCKED:** mecanismo e política de recovery não definidos.

## Restrições

- trabalho demorado não bloqueia o ciclo de recebimento Telegram;
- estados: `queued`, `running`, `completed`, `failed`, `cancelled`, `timeout`.

## Critérios de aceite

- criação do job é separada da execução;
- progresso pode atualizar a mesma mensagem;
- faltam concorrência, fila, retry, cancelamento e restart para chegar a `refined`.

## Implementação relacionada

Prevista para os pacotes internos de jobs, fila e workers.
