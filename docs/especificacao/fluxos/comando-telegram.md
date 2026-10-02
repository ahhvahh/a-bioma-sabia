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
6. o adaptador converte a solicitação em comando interno;
7. o Command Router resolve a operação cadastrada;
8. a operação produz resultado imediato ou cria job;
9. o adaptador persiste a resposta como entrega `pending`;
10. o adaptador envia a resposta pelo Telegram;
11. após sucesso da Bot API, persiste o `message_id` retornado e marca a entrega como `delivered`;
12. auditoria registra a operação sem segredos.

## Fluxos alternativos

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

A política de retry da leitura por `getUpdates` ainda precisa ser definida.

## Resultado

Resposta controlada ao usuário ou referência de job criado, com estado de entrega persistido.

## Critérios de aceite

- autorização antecede execução;
- cliente não acessa comandos de outro cliente;
- falha antes da persistência não confirma o update;
- update persistido pode ser recuperado localmente após reinício;
- o mesmo `update_id` persistido para um cliente não gera execução duplicada;
- uma resposta não é `delivered` sem confirmação e persistência do `message_id`;
- `retry_after` impede retry antecipado;
- falha ambígua mantém a resposta pendente;
- falta fechar retry de leitura da Bot API para `refined`.

## Implementação relacionada

Adaptador Telegram, segurança e Command Router.
