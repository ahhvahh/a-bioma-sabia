# Registro e execução de scripts

![MOD](https://img.shields.io/badge/MOD-MOD--0003-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Resolver operações locais por identificador permitido e executá-las com timeout e captura de resultado.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0004 — Registro explícito de scripts](../../adr/execucao/registro-explicito-de-scripts.md)
- [CTR-0002 — Execução de script](../contratos/execucao-script.md)

## Responsabilidades

- carregar definições de scripts;
- resolver identificador lógico;
- aplicar timeout cadastrado;
- executar somente definição resolvida pelo registro;
- capturar stdout, stderr, exit code e duração;
- fornecer resultado para comando, job ou scheduler.

## Entradas

Identificador de script já autorizado e seus parâmetros permitidos pelo contrato da operação.

## Saídas

Resultado de execução conforme CTR-0002.

## Interfaces e contratos

- [CTR-0002 — Execução de script](../contratos/execucao-script.md)

## Persistência

Registro proveniente de configuração. Nenhum estado operacional persistente foi definido para o módulo.

## Restrições

- proibir caminho arbitrário vindo do usuário;
- proibir concatenação de mensagem em `bash -c`;
- ferramentas concretas não devem contaminar o Command Router.

## Critérios de aceite

- timeout é detectável;
- stdout e stderr são separados;
- exit code é preservado;
- falta definir invocação, working directory, ambiente permitido e limite de saída antes de `refined`.

## Implementação relacionada

Prevista para os pacotes internos de scripts e executor.
