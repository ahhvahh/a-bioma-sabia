# ASM-001 — A instalação possui conectividade de saída suficiente para acessar a Telegram Bot API.

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Premissa

A instalação possui conectividade de saída suficiente para acessar a Telegram Bot API.

## Motivo

A condição é necessária para a operação das capacidades dependentes na baseline atual.

## Evidência atual

Requisito operacional decorrente do uso de long polling e envio de respostas.

## Risco se estiver incorreta

As capacidades dependentes podem não operar conforme os fluxos documentados.

## Validação

**Precisa de Discovery:** não  
**Discovery:** -

## Itens afetados

- Ver [Scope](../../scope/README.md).
