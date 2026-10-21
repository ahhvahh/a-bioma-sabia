# INT-001 — Telegram Bot API

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Propósito

Receber updates e entregar mensagens/mídias.

## Origem e destino

**Origem:** Sabiá  
**Destino:** Telegram Bot API  
**Direção:** bidirecional  
**Obrigatória:** sim

## Entrada

Updates e confirmações do serviço remoto

## Saída

Chamadas de leitura, envio e edição

## Responsabilidades

O Sabiá assume somente as responsabilidades de integração explicitamente descritas; a lógica interna do sistema externo permanece fora de sua fronteira.

## Fluxos relacionados

- FLW-001
- FLW-002
- FLW-003
- FLW-004
- FLW-005
- FLW-006
- FLW-007

## Conhecimento e Discovery

**Estado:** DOCUMENTED  
**Discovery:** -

## Atualização 2026-10-21 — compatibilidade

Base oficial: Bot API 10.3 (24/08/2026). Entrada `getUpdates` com mensagens e callbacks; saída `sendMessage`, `InlineKeyboardMarkup`, `answerCallbackQuery`, `editMessageText`, `sendPhoto`, `sendVideo`, `sendDocument`. Aceite de update exige JSON recuperável e avanço do cursor no mesmo commit; nenhuma confirmação antes disso. Limite operacional total 50 MB; download por `getFile` da API pública até 20 MB e fotos sujeitas a limites próprios. Fonte: https://core.telegram.org/bots/api.
