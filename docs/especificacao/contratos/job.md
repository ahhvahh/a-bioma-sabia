# Job

![CTR](https://img.shields.io/badge/CTR-CTR--0003-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir os estados e informações mínimas de uma operação assíncrona.

## Dependências

- [MOD-0004 — Jobs](../modulos/jobs.md)
- [ADR-0006 — Jobs assíncronos](../../adr/processamento/jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)

## Tipo

`interface`

## Entrada

Solicitação de operação demorada com contexto de resposta. Para Telegram, a origem precisa permitir correlacionar:
- `chat_id`;
- `message_id`;
- `job_id`.

## Saída

Estados permitidos:
- `queued`;
- `running`;
- `completed`;
- `failed`;
- `cancelled`;
- `timeout`.

O job pode publicar progresso e, ao concluir, produzir resultado textual ou arquivo.

## Erros

Falha, cancelamento e timeout são estados explícitos, não exceções invisíveis ao usuário.

## Regras e restrições

- criação do job responde sem aguardar conclusão;
- trabalho é executado por worker;
- mudanças de estado devem ser observáveis;
- estado inválido deve ser rejeitado.

## Compatibilidade

A correlação de resposta deve permitir outros transportes no futuro, sem exigir que o modelo inteiro seja exclusivamente Telegram.

## Critérios de aceite

Antes de `refined`, definir:
- tipo e geração de `job_id`;
- capacidade da fila;
- concorrência de workers;
- política de retry;
- semântica de cancelamento;
- persistência/recovery;
- transições exatas permitidas.
