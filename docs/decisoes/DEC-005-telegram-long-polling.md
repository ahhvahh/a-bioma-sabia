<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# DEC-005 — Telegram Long Polling

**ID:** `DEC-005`  
**ID histórico:** `ADR-0005`  
**Estado documental de origem:** `refined`  
**Versão histórica registrada:** `1`  
**Implementação:** não verificada nesta revisão

## Decisão

Usar Telegram Long Polling no primeiro MVP.

## Contexto e problema

O primeiro MVP deve receber comandos Telegram sem expor portas públicas no servidor.

Escolher o mecanismo de recebimento de updates da Telegram Bot API.

## Restrições

- não expor endpoint público de webhook no MVP;
- manter clientes independentes;
- usar comportamento compatível com a Bot API oficial.

## Alternativas consideradas

### Webhook

Exige endpoint HTTPS acessível pelo Telegram.

### Long polling com `getUpdates`

Permite ao serviço iniciar conexões de saída para buscar updates.

## Justificativa

Atende ao requisito explícito de não expor porta pública e ao escopo definido para o MVP.

## Consequências

- cada cliente habilitado precisa manter seu ciclo de obtenção de updates;
- webhook e long polling não podem ser usados simultaneamente para o mesmo bot;
- confirmação e avanço de `offset` precisam ser definidos no contrato operacional do adaptador.

## Critérios de validação

- o serviço recebe updates por `getUpdates`;
- nenhum listener HTTP público é necessário para o Telegram;
- o comportamento segue a documentação oficial: https://core.telegram.org/bots/api#getupdates

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](DEC-002-clientes-telegram-isolados.md)
- [ADR-0003 — Core independente do Telegram](DEC-003-core-independente-telegram.md)

## Limitações da revisão

O offset deve respeitar a aceitação durável dos updates; consulte [entidade inbound_update](../persistencia/entidades/inbound-update.md). Não há comprovação de teste real de long polling.

A revisão documental não demonstra implementação, execução de testes ou aprovação de novas regras.

## Fonte e rastreabilidade

Documento histórico preservado: [`ADR-0005` original](../adr/telegram/long-polling.md). O identificador e a versão original foram mantidos para rastrear a decisão, sem renumerar outros ADRs.
