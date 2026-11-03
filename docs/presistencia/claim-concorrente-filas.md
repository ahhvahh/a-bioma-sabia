# Claim concorrente de filas persistentes

**ID:** PST-0003  
**Status:** refined

## Objetivo

Definir como jobs, mensagens de saída e transmissões de mídia são reservados por um único consumidor concorrente no PostgreSQL antes do processamento externo.

## Dependências

- [ADR-0009 — Persistência do estado operacional em PostgreSQL](../../adr/persistencia/estado-operacional.md)
- [CTR-0003 — Job](../contratos/job.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [FLW-0010 — Consumo do estado operacional](../fluxos/consumo-estado-operacional.md)
- [PST-0001 — Tabelas do estado operacional](tabelas-estado-operacional.md)

## Regra

Filas persistentes com múltiplos consumidores não podem ser consumidas por um `SELECT` comum seguido posteriormente por alteração de estado.

O consumidor deve solicitar o próximo item por uma rotina armazenada no PostgreSQL que, dentro da mesma transação curta:

1. localiza um registro elegível;
2. bloqueia a linha com `FOR UPDATE SKIP LOCKED`;
3. altera o registro para seu estado intermediário de processamento;
4. persiste a alteração;
5. retorna o item reservado ao consumidor;
6. realiza `COMMIT` antes de qualquer processamento externo.

O processamento de script, chamada à Telegram Bot API ou streaming de mídia ocorre somente depois do commit. Nenhum lock de linha deve permanecer aberto durante processamento externo.

O nível normal de isolamento é `READ COMMITTED`. A exclusividade do claim não depende apenas do nível de isolamento: ela é garantida pelo lock da linha e pela transição de estado realizados na mesma transação.

## Estados de reserva

| Fila | Estado elegível | Estado após claim |
|---|---|---|
| `job` | `queued` | `running` |
| `outbound_message` | `pending` | `sending` |
| `media_transmission` | `pending` | `transmitting` |

Para `media_transmission`, registros `transmitting` encontrados no recovery continuam sujeitos às regras de retomada de CTR-0007; eles não são tratados como um novo claim concorrente sem antes se tornarem elegíveis segundo o fluxo de recovery.

## Atomicidade

Seleção, bloqueio e transição de estado pertencem à mesma transação.

Se a transação falhar ou sofrer rollback, o registro permanece no estado anterior e pode ser reclamado por outro consumidor.

Depois do commit, outro consumidor não pode obter o mesmo registro pelo filtro do estado anterior.

A rotina deve retornar no máximo um registro por claim. O consumidor solicita outro item somente após receber o resultado do claim anterior.

## Mensagens de saída

O estado de `outbound_message` passa a ser:

`pending → sending → delivered | pending | failed`

- `pending → sending`: claim atômico antes da chamada ao transporte;
- `sending → delivered`: confirmação remota recebida e persistida;
- `sending → pending`: falha temporária, resultado ambíguo ou retry permitido;
- `sending → failed`: falha permanente;
- no startup, mensagem encontrada em `sending` volta para `pending` antes da retomada dos entregadores.

Essa recuperação preserva a semântica `at-least-once`: uma entrega remota concluída cuja confirmação local tenha sido perdida pode ser repetida.

## Jobs

O claim de job realiza `queued → running` antes de devolver o job ao worker.

A mudança de `job.status` e a inclusão correspondente em `job_state_history` devem ocorrer na mesma transação do claim.

Depois do commit, o worker executa a operação fora da transação de claim.

Recovery de jobs encontrados em `running` continua seguindo CTR-0003: eles não são reexecutados silenciosamente e passam para `failed/service_restart`.

## Transmissões de mídia

O claim de mídia realiza `pending → transmitting` antes de iniciar o envio.

O streaming ocorre fora da transação de claim. O estado `transmitting` permanece persistido durante a tentativa e segue CTR-0007 para confirmação, falha e recovery.

## Concorrência

Consumidores concorrentes podem executar as rotinas de claim simultaneamente.

Uma linha já bloqueada por outro claim é ignorada por `SKIP LOCKED`, permitindo que outro consumidor obtenha outro item elegível sem aguardar o término daquela transação.

Workers não devem implementar uma segunda estratégia independente de reserva em memória como fonte de exclusividade. O PostgreSQL é a autoridade para o claim das filas persistentes.

## Critérios de aceite

- dois workers concorrentes não recebem o mesmo registro pelo mesmo estado elegível;
- seleção e transição de estado são atômicas;
- rollback não deixa o registro falsamente reservado;
- locks não permanecem abertos durante operações externas;
- job reclamado está `running` antes da execução;
- mensagem reclamada está `sending` antes do envio;
- mídia reclamada está `transmitting` antes do streaming;
- recovery de mensagens em `sending` retorna o item a `pending`;
- recovery de jobs e transmissões preserva CTR-0003 e CTR-0007.
