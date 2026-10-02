# Adaptador Telegram

![MOD](https://img.shields.io/badge/MOD-MOD--0002-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Integrar cada cliente configurado com a Telegram Bot API sem acoplar o Core ao protocolo externo.

## Dependências

- [DSG-0001 — Contexto do Sabiá](../../desenho/contexto-sabia.md)
- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0005 — Telegram Long Polling](../../adr/telegram/long-polling.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../../adr/seguranca/menor-privilegio-e-autorizacao.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)

## Responsabilidades

- iniciar long polling por cliente habilitado;
- converter update em comando interno;
- aplicar o contexto correto do cliente;
- encaminhar identidade para autorização;
- converter respostas internas em mensagens Telegram;
- editar mensagem de progresso de job;
- enviar arquivo final quando existir;
- registrar no estado operacional as correlações e entregas pendentes necessárias ao recovery.

## Entradas

Updates recebidos por `getUpdates`.

## Saídas

Comandos internos e chamadas de envio/edição de mensagens.

## Interfaces e contratos

- [CTR-0001 — Comando interno](../contratos/comando-interno.md)
- [CTR-0003 — Job](../contratos/job.md)

## Persistência

O estado operacional usa SQLite conforme ADR-0009. Requisições e respostas pendentes devem permanecer correlacionadas ao destino para que, após reinício, mensagens prontas e ainda não entregues voltem à etapa de envio.

## Restrições

- long polling e webhook não são usados simultaneamente para o mesmo cliente;
- tokens não aparecem em logs;
- nenhum teste depende da API real.

## Critérios de aceite

- clientes habilitados funcionam independentemente;
- update é convertido sem expor tipos Telegram ao restante do Core além da fronteira;
- mensagens de jobs podem ser editadas;
- uma mensagem persistida como pendente pode voltar à etapa de envio depois de reinício;
- política detalhada de offset/retry ainda precisa ser fechada antes de `refined`.

## Implementação relacionada

Prevista para o pacote interno de integração Telegram.
