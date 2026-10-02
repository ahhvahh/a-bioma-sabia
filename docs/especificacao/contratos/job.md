# Job

![CTR](https://img.shields.io/badge/CTR-CTR--0003-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Definir identidade, estados, fila, concorrência, cancelamento, recovery e entrega de respostas de uma operação assíncrona.

## Dependências

- [MOD-0004 — Jobs](../modulos/jobs.md)
- [ADR-0006 — Jobs assíncronos](../../adr/processamento/jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)

## Tipo

`interface`

## Entrada

Toda solicitação assíncrona deve persistir, no mínimo:

- `job_id`;
- `request_id`;
- `client_id`;
- identidade do solicitante;
- `reply_context` com transporte e destino de resposta;
- operação solicitada;
- estado;
- timestamps relevantes;
- referência à mensagem Telegram quando existir.

Para Telegram, a correlação operacional preserva `client_id`, `reply_context.destination_id` e `message_id` quando aplicável. O `request_id` permanece estável entre comando, job, processador e entrega.

## Identidade

`job_id` é um inteiro sequencial gerado pelo SQLite.

## Fila e concorrência

- a fila é persistida no SQLite;
- `jobs.max_workers` define quantos jobs podem estar `running` simultaneamente;
- `jobs.max_pending` define quantos jobs `queued` podem aguardar execução;
- ambos são parâmetros obrigatórios, inteiros e maiores que zero;
- ao atingir `max_pending`, nova solicitação de job é rejeitada com erro controlado de capacidade.

## Estados e transições

Estados permitidos:

- `queued`;
- `running`;
- `completed`;
- `failed`;
- `cancelled`;
- `timeout`.

Transições permitidas:

- `queued → running`;
- `queued → cancelled`;
- `running → completed`;
- `running → failed`;
- `running → cancelled`;
- `running → timeout`.

Estados finais não retornam para estados de execução.

## Retry

Não existe retry automático por padrão.

Uma operação só pode ser repetida automaticamente quando estiver explicitamente configurada como repetível e possuir `max_retries > 0`. Retry cria uma nova tentativa associada ao mesmo job sem apagar o histórico da tentativa anterior.

## Cancelamento

- job `queued`: removido da execução futura e marcado `cancelled`;
- job `running`: o worker solicita cancelamento pelo contexto de execução e encerra o processo associado;
- cancelamento confirmado termina em `cancelled`;
- falha técnica ao cancelar termina em `failed` com motivo registrado.

## Recovery

No startup:

- `queued` permanece `queued` e volta a ser elegível para execução;
- job encontrado em `running` é marcado `failed` com motivo `service_restart`;
- não há reexecução automática de job que estava `running`;
- respostas prontas e ainda não entregues voltam para a etapa de envio.

## Progresso de processadores assíncronos

Quando o job for executado por processador conforme CTR-0005:

- evento `loading` é progresso intermediário e não altera o estado terminal;
- evento `finally` encerra semanticamente o processamento normal;
- todo evento precisa usar o mesmo `request_id` do job;
- término do processo sem `finally` não equivale automaticamente a `completed`.

## Entrega da resposta

Toda resposta pertence ao `client_id` e ao destino persistido da requisição.

A conclusão do processamento não equivale à conclusão da entrega. Enquanto uma resposta não tiver sido enviada ao cliente/destino correto, ela permanece pendente no estado operacional.

## Visibilidade

Consultas normais de `/jobs` e `/job <id>` retornam somente jobs pertencentes ao mesmo cliente lógico e ao mesmo solicitante autorizado. Não existe visibilidade administrativa global implícita.

## Erros

- fila cheia;
- operação inválida;
- falha de execução;
- cancelamento;
- timeout;
- falha de entrega;
- job inexistente ou não visível ao solicitante.

## Regras e restrições

- criação do job responde sem aguardar sua conclusão;
- mudanças de estado devem ser persistidas;
- resposta final precisa ser encaminhada ao cliente e destino correlacionados;
- `request_id` deve permanecer rastreável em progresso e resultado final;
- estado inválido ou transição inválida deve ser rejeitado;
- histórico de mudança de estado não deve ser apagado pela atualização do estado atual.

## Compatibilidade

A correlação de resposta deve permitir outros transportes futuramente sem tornar o modelo exclusivamente Telegram.

## Critérios de aceite

- IDs são persistentes e sequenciais;
- fila respeita `max_pending`;
- concorrência respeita `max_workers`;
- queued sobrevive ao restart;
- running interrompido por restart vira `failed/service_restart`;
- não existe retry automático sem autorização explícita;
- cancelamento respeita o estado atual;
- resposta permanece pendente até ser entregue ao cliente/destino correto;
- `loading` não conclui o job e `finally` é a finalização semântica normal para processadores CTR-0005.
