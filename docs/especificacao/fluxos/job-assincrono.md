# Job assíncrono

![FLW](https://img.shields.io/badge/FLW-FLW--0002-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Executar uma operação demorada sem bloquear novos comandos e entregar o resultado ao cliente e destino corretos.

## Dependências

- [MOD-0004 — Jobs](../modulos/jobs.md)
- [CTR-0003 — Job](../contratos/job.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)

## Gatilho

Command Router identifica uma operação assíncrona válida.

## Pré-condições

- operação válida e autorizada;
- `client_id` e destino de resposta disponíveis;
- fila abaixo de `jobs.max_pending`.

## Fluxo principal

1. Job Manager cria e persiste o job;
2. o job entra em `queued`;
3. o cliente envia confirmação imediata ao destino de origem;
4. um worker disponível muda o job para `running`;
5. Worker executa a operação;
6. progresso, quando existir, é encaminhado pelo cliente correto;
7. Worker conclui em estado final;
8. o resultado é persistido como resposta pendente;
9. adaptador do `client_id` correspondente envia a resposta ao destino persistido;
10. somente depois da entrega o estado de entrega é registrado como concluído.

## Fluxos alternativos

### Fila cheia

Rejeitar a criação com erro controlado de capacidade.

### Restart com job queued

O job retorna à fila.

### Restart com job running

Marcar como `failed/service_restart`; não reexecutar automaticamente.

### Restart com resposta pendente

Recolocar a resposta na etapa de envio.

### Cancelamento

Aplicar as regras de CTR-0003 conforme o estado atual.

## Falhas e tratamento

Falha de processamento e falha de entrega são registradas separadamente. Um resultado processado não é considerado entregue enquanto o cliente não concluir a tentativa de envio.

## Resultado

Job possui estado final persistido e sua resposta permanece rastreável até a entrega.

## Critérios de aceite

- fila e concorrência respeitam configuração;
- queued sobrevive ao restart;
- running interrompido não reinicia silenciosamente;
- resposta é enviada pelo cliente correto;
- resposta não entregue permanece pendente.

## Implementação relacionada

JobManager, JobQueue, Worker, SQLite e adaptador Telegram.
