# Comando Telegram

![FLW](https://img.shields.io/badge/FLW-FLW--0001-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Processar um comando recebido por um cliente Telegram até sua resposta, preservando autorização e isolamento.

## Dependências

- [MOD-0002 — Adaptador Telegram](../modulos/telegram.md)
- [MOD-0006 — Segurança e autorização](../modulos/seguranca.md)
- [MOD-0001 — Core e Command Router](../modulos/core-command-router.md)
- [CTR-0001 — Comando interno](../contratos/comando-interno.md)

## Gatilho

Update recebido por long polling em cliente habilitado.

## Pré-condições

- cliente habilitado;
- token disponível por fonte permitida;
- update pertence ao cliente correto.

## Fluxo principal

1. o adaptador recebe update;
2. identifica usuário, chat e conteúdo relevante;
3. persiste transacionalmente o update e sua correlação com o cliente usando `update_id` como chave idempotente;
4. somente após a persistência bem-sucedida, o ciclo de polling pode avançar para `offset = maior update_id persistido + 1`;
5. a autorização valida whitelist do cliente e, quando configurado, chat;
6. o adaptador gera `request_id`, normaliza `principal_id`, define `client_id`, `received_at` e `reply_context = { transport: "telegram", destination_id: <chat normalizado> }`, tokeniza os argumentos e constrói o comando interno com `arguments: string[]`, sem aplicar validação semântica específica da operação;
7. o Command Router resolve a operação cadastrada e a operação valida quantidade, formato e domínio dos argumentos;
8. a operação produz resultado imediato ou cria job;
9. o adaptador persiste a resposta como entrega `pending`;
10. o adaptador envia a resposta pelo Telegram;
11. após sucesso da Bot API, persiste o `message_id` retornado e marca a entrega como `delivered`;
12. auditoria registra a operação sem segredos.

## Fluxos alternativos

### Falha temporária de leitura por getUpdates

1. timeout de transporte, falha de rede ou resposta `5xx` não altera o `offset`;
2. o cliente aplica backoff exponencial `1s → 2s → 4s → 8s → 16s → 30s`;
3. nova falha temporária avança uma etapa até o máximo de `30s`;
4. uma leitura bem-sucedida, ainda que sem updates, reseta o backoff para `1s`;
5. se o Telegram retornar `429` com `retry_after`, o cliente aguarda pelo menos o intervalo informado antes da próxima leitura;
6. falhas e backoff de um cliente não interferem nos demais.

### Falha de autenticação ou autorização no polling

1. erro de autenticação/autorização, incluindo `401` ou `403`, suspende o polling daquele cliente;
2. o erro é registrado para observabilidade;
3. os demais clientes continuam operando;
4. nenhuma alteração de `offset` ocorre.

### Falha antes da persistência do update

1. o update não é considerado aceito;
2. o `offset` não avança além desse `update_id`;
3. uma leitura posterior pode receber novamente o update.

### Update já persistido

1. o adaptador identifica a duplicidade pelo `update_id` dentro do cliente correspondente;
2. nenhuma segunda execução da mesma requisição é criada;
3. o estado local já existente continua sendo a fonte para processamento ou recovery.

### Retry de envio solicitado pelo Telegram

1. a entrega permanece `pending`;
2. se a resposta contiver `retry_after`, nenhuma nova tentativa pode ocorrer antes do intervalo informado;
3. depois do intervalo, a mesma entrega volta a ser elegível para envio.

### Falha temporária ou ambígua de envio

1. timeout, desconexão ou erro temporário sem confirmação mantém a entrega `pending`;
2. a aplicação registra a tentativa;
3. a entrega pode ser reenviada;
4. eventual duplicidade de mensagem é aceita para preservar semântica `at-least-once`.

### Falha permanente de envio

1. a entrega é marcada como `failed`;
2. código e descrição do erro são preservados;
3. não há retry automático dessa entrega sem nova ação explicitamente definida.

### Restart com resposta pendente

1. respostas `pending` retornam à etapa de envio;
2. respostas `delivered` não são reenviadas;
3. respostas `failed` permanecem encerradas.

### Usuário ou chat não autorizado

1. a operação não chega ao executor;
2. a tentativa é registrada conforme política de auditoria.

### Comando desconhecido

1. o Router retorna erro controlado;
2. nenhuma execução local ocorre.

## Falhas e tratamento

A entrega usa semântica `at-least-once`. Sucesso só é confirmado após retorno positivo da Bot API e persistência do `message_id`. Falha ambígua permanece pendente e pode resultar em duplicidade eventual.

A leitura por `getUpdates` usa backoff exponencial limitado a `30s` para falhas temporárias. `429` respeita `retry_after`; falhas de autenticação/autorização suspendem somente o cliente afetado. Nenhuma falha de polling altera o `offset`.

## Resultado

Resposta controlada ao usuário ou referência de job criado, com estado de entrega persistido.

## Critérios de aceite

- autorização antecede execução;
- comando interno possui `request_id`, `client_id`, `principal_id`, `reply_context` e `received_at` normalizados pelo adaptador;
- `reply_context.destination_id` representa o chat de resposta como string opaca ao Core;
- argumentos chegam ao Core como `string[]` já tokenizado pelo adaptador;
- validação semântica dos argumentos ocorre na operação correspondente;
- cliente não acessa comandos de outro cliente;
- falha antes da persistência não confirma o update;
- update persistido pode ser recuperado localmente após reinício;
- o mesmo `update_id` persistido para um cliente não gera execução duplicada;
- uma resposta não é `delivered` sem confirmação e persistência do `message_id`;
- `retry_after` impede retry antecipado;
- falha ambígua mantém a resposta pendente;
- falha temporária de polling usa backoff exponencial até `30s`;
- leitura bem-sucedida reseta o backoff para `1s`;
- `429` respeita `retry_after`;
- erro de autenticação/autorização suspende somente o cliente afetado;
- falha de polling não altera o `offset`.

## Implementação relacionada

Adaptador Telegram, segurança e Command Router.
