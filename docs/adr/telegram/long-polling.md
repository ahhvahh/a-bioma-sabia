# Telegram Long Polling

![ADR](https://img.shields.io/badge/ADR-ADR--0005-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

O primeiro MVP deve receber comandos Telegram sem expor portas públicas no servidor.

## Problema

Escolher o mecanismo de recebimento de updates da Telegram Bot API.

## Restrições

- não expor endpoint público de webhook no MVP;
- manter clientes independentes;
- usar comportamento compatível com a Bot API oficial.

## Opções consideradas

### Webhook

Exige endpoint HTTPS acessível pelo Telegram.

### Long polling com `getUpdates`

Permite ao serviço iniciar conexões de saída para buscar updates.

## Decisão

Usar Telegram Long Polling no primeiro MVP.

## Justificativa

Atende ao requisito explícito de não expor porta pública e ao escopo definido para o MVP.

## Consequências

- cada cliente habilitado precisa manter seu ciclo de obtenção de updates;
- webhook e long polling não podem ser usados simultaneamente para o mesmo bot;
- confirmação e avanço de `offset` precisam ser definidos no contrato operacional do adaptador.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](multiplos-clientes-isolados.md)
- [ADR-0003 — Core independente do Telegram](../arquitetura/core-independente-do-telegram.md)

## Critérios de validação

- o serviço recebe updates por `getUpdates`;
- nenhum listener HTTP público é necessário para o Telegram;
- o comportamento segue a documentação oficial: https://core.telegram.org/bots/api#getupdates
