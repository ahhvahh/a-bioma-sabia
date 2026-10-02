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
3. a autorização valida whitelist do cliente e, quando configurado, chat;
4. o adaptador converte a solicitação em comando interno;
5. o Command Router resolve a operação cadastrada;
6. a operação produz resultado imediato ou cria job;
7. o adaptador converte o resultado em resposta Telegram;
8. auditoria registra a operação sem segredos.

## Fluxos alternativos

### Usuário ou chat não autorizado

1. a operação não chega ao executor;
2. a tentativa é registrada conforme política de auditoria.

### Comando desconhecido

1. o Router retorna erro controlado;
2. nenhuma execução local ocorre.

## Falhas e tratamento

Falhas de Bot API, política de retry, avanço de offset e comportamento após reinício ainda precisam ser definidos.

## Resultado

Resposta controlada ao usuário ou referência de job criado.

## Critérios de aceite

- autorização antecede execução;
- cliente não acessa comandos de outro cliente;
- falta fechar retry/offset para `refined`.

## Implementação relacionada

Adaptador Telegram, segurança e Command Router.
