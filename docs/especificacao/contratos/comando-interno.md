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

### Argumentos

Os argumentos entram no Core como:

`arguments: string[]`

Regras:

- o adaptador de entrada é responsável por transformar a representação recebida no transporte em uma lista ordenada de strings;
- ausência de argumentos é representada por lista vazia;
- o adaptador não aplica validação semântica específica da operação;
- quantidade, formato, domínio, obrigatoriedade e significado de cada argumento pertencem à operação resolvida pelo Command Router;
- o Core não recebe texto bruto do Telegram para interpretar a sintaxe do comando;
- nenhum objeto específico da Telegram Bot API pode ser carregado em `arguments`.

Os tipos concretos dos demais campos, limites e estrutura completa do schema ainda precisam ser definidos.

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
- comando não carrega caminho de script arbitrário;
- tokenização é responsabilidade do adaptador;
- validação semântica dos argumentos é responsabilidade da operação correspondente.

## Compatibilidade

Novos transportes devem conseguir produzir o mesmo comando conceitual e entregar argumentos como `string[]`, independentemente da sintaxe original do transporte.

## Critérios de aceite

- o Router funciona em teste sem Telegram;
- argumentos chegam ao Core como `string[]` já tokenizado;
- adaptadores não precisam conhecer as regras semânticas específicas de cada operação;
- operações validam seus próprios argumentos;
- ainda falta fechar os demais campos, tipos, limites e estruturas de resultado/erro antes de `refined`.
