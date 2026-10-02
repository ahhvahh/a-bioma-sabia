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
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)

## Responsabilidades

- iniciar long polling por cliente habilitado;
- converter update em comando interno;
- aplicar o contexto correto do cliente;
- encaminhar identidade para autorização;
- converter respostas internas em mensagens Telegram;
- editar mensagem de progresso de job;
- enviar resposta/arquivo final pelo mesmo cliente e destino correlacionado à requisição;
- registrar correlações e entregas pendentes;
- durante shutdown, avisar todos os clientes ativos por seus destinos autorizados conhecidos.

## Entradas

Updates recebidos por `getUpdates`.

## Saídas

Comandos internos e chamadas de envio/edição de mensagens.

## Interfaces e contratos

- [CTR-0001 — Comando interno](../contratos/comando-interno.md)
- [CTR-0003 — Job](../contratos/job.md)

## Persistência

SQLite mantém a correlação entre requisição, cliente e destino de resposta. Mensagens prontas e não entregues voltam à etapa de envio após reinício.

## Restrições

- long polling e webhook não são usados simultaneamente para o mesmo cliente;
- tokens não aparecem em logs;
- nenhum teste depende da API real;
- uma resposta de um cliente não pode ser entregue usando identidade de outro cliente.

## Critérios de aceite

- clientes habilitados funcionam independentemente;
- resposta é enviada pelo cliente que recebeu a requisição;
- todos os clientes ativos recebem tentativa de aviso no shutdown;
- mensagem pendente pode voltar à etapa de envio após reinício;
- política detalhada de offset/retry ainda precisa ser fechada antes de `refined`.

## Implementação relacionada

Prevista para o pacote interno de integração Telegram.
