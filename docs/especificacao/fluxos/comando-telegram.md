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
9. o adaptador converte o resultado em resposta Telegram;
10. auditoria registra a operação sem segredos.

## Fluxos alternativos

### Falha antes da persistência do update

1. o update não é considerado aceito;
2. o `offset` não avança além desse `update_id`;
3. uma leitura posterior pode receber novamente o update.

### Update já persistido

1. o adaptador identifica a duplicidade pelo `update_id` dentro do cliente correspondente;
2. nenhuma segunda execução da mesma requisição é criada;
3. o estado local já existente continua sendo a fonte para processamento ou recovery.

### Usuário ou chat não autorizado

1. a operação não chega ao executor;
2. a tentativa é registrada conforme política de auditoria.

### Comando desconhecido

1. o Router retorna erro controlado;
2. nenhuma execução local ocorre.

## Falhas e tratamento

Falhas de Bot API, política de retry de leitura/envio e recovery de respostas pendentes ainda precisam ser definidos.

## Resultado

Resposta controlada ao usuário ou referência de job criado.

## Critérios de aceite

- autorização antecede execução;
- cliente não acessa comandos de outro cliente;
- falha antes da persistência não confirma o update;
- update persistido pode ser recuperado localmente após reinício;
- o mesmo `update_id` persistido para um cliente não gera execução duplicada;
- falta fechar retry da Bot API e recovery de respostas para `refined`.

## Implementação relacionada

Adaptador Telegram, segurança e Command Router.
