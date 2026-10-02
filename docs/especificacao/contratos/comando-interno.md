# Comando interno

![CTR](https://img.shields.io/badge/CTR-CTR--0001-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir a fronteira entre adaptadores de entrada e o Command Router sem transportar dependência direta da Telegram Bot API para o Core.

## Dependências

- [MOD-0001 — Core e Command Router](../modulos/core-command-router.md)
- [MOD-0002 — Adaptador Telegram](../modulos/telegram.md)
- [ADR-0003 — Core independente do Telegram](../../adr/arquitetura/core-independente-do-telegram.md)

## Tipo

`interface`

## Entrada

O comando interno deve representar, no mínimo:
- cliente lógico que recebeu a solicitação;
- identidade já submetida à autorização;
- identificador do comando;
- argumentos do comando, quando permitidos;
- contexto suficiente para direcionar uma resposta;
- referência a anexos quando o comando aceitar mídia.

Os tipos concretos e limites ainda não estão definidos.

## Saída

Resultado interno passível de conversão pelo adaptador: resposta textual, criação/referência de job, erro controlado e, quando aplicável, arquivo de resultado.

## Erros

- comando desconhecido;
- argumentos inválidos;
- operação não disponível para o cliente;
- falha do executor.

## Regras e restrições

- não expor objetos da Bot API como contrato do domínio;
- autorização ocorre antes da execução;
- comando não carrega caminho de script arbitrário.

## Compatibilidade

Novos transportes devem conseguir produzir o mesmo comando conceitual.

## Critérios de aceite

- o Router funciona em teste sem Telegram;
- falta fechar schema/tipos/limites antes de `refined`.
