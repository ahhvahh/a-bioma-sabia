# Execução de script

![CTR](https://img.shields.io/badge/CTR-CTR--0002-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Padronizar o contrato entre Script Registry, executor, jobs e scheduler.

## Dependências

- [MOD-0003 — Registro e execução de scripts](../modulos/scripts.md)
- [ADR-0004 — Registro explícito de scripts](../../adr/execucao/registro-explicito-de-scripts.md)

## Tipo

`interface`

## Entrada

Definição previamente cadastrada contendo, no mínimo:
- identificador lógico;
- caminho configurado;
- timeout configurado.

O usuário remoto não fornece o caminho.

## Saída

O executor deve produzir:
- stdout;
- stderr;
- exit code;
- tempo de execução;
- indicação de timeout.

Para scripts de monitoramento, a convenção preferencial é:
- `0 = OK`;
- `1 = WARNING`;
- `2 = CRITICAL`;
- `3 = UNKNOWN`.

Retorno estruturado JSON pode ser adicionado futuramente sem tornar texto livre a única fonte de estado.

## Erros

- identificador não encontrado;
- falha ao iniciar processo;
- timeout;
- término com exit code não esperado;
- erro interno de execução.

## Regras e restrições

- nunca construir `bash -c` com mensagem Telegram;
- nunca aceitar caminho arbitrário do usuário;
- preservar stdout e stderr separadamente.

## Compatibilidade

O contrato deve permitir executores futuros além de scripts sem alterar o Command Router.

## Critérios de aceite

Antes de `refined`, ainda é necessário definir:
- invocação do arquivo cadastrado;
- diretório de trabalho;
- variáveis de ambiente herdadas/permitidas;
- limite de stdout/stderr;
- tratamento de processo filho no timeout.
