# Registro e execução de scripts

![MOD](https://img.shields.io/badge/MOD-MOD--0003-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Resolver operações locais por identificador permitido e executá-las de forma controlada conforme CTR-0002.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0004 — Registro explícito de scripts](../../adr/execucao/registro-explicito-de-scripts.md)
- [CTR-0002 — Execução de script](../contratos/execucao-script.md)
- [CFG-0001 — Modelo de configuração](../configuracao/modelo-configuracao.md)

## Responsabilidades

- carregar definições de scripts;
- resolver identificador lógico;
- validar a definição cadastrada;
- aplicar timeout cadastrado;
- executar somente definição resolvida pelo registro;
- permitir execuções simultâneas independentes do mesmo script;
- capturar stdout, stderr, exit code, duração, timeout, cancelamento e truncamento nas execuções convencionais;
- quando o script atuar como processador CTR-0005, entregar `request_id` por stdin e tratar stdout como stream JSON Lines de eventos;
- encerrar o grupo de processos da execução em timeout/cancelamento;
- fornecer resultado para comando, job ou scheduler.

## Entradas

Identificador de script já autorizado e argumentos permitidos pela operação.

Caminho do executável, interpretador, diretório de trabalho e política de ambiente são resolvidos a partir da configuração e não da solicitação remota.

## Saídas

Resultado de execução conforme [CTR-0002 — Execução de script](../contratos/execucao-script.md).

## Interfaces e contratos

- [CTR-0002 — Execução de script](../contratos/execucao-script.md)

## Persistência

O registro é proveniente de configuração.

O módulo não precisa persistir estado próprio de execução; quando a execução pertence a um job, estado e recovery são responsabilidade do subsistema de jobs.

## Restrições

- proibir caminho arbitrário vindo do usuário;
- proibir concatenação de mensagem em `bash -c` ou equivalente;
- somente variáveis explicitamente permitidas podem ser herdadas;
- stdout e stderr possuem limite de 1 MiB cada por execução;
- uma execução não pode cancelar ou contaminar o estado de outra;
- ferramentas concretas não devem contaminar o Command Router.

## Critérios de aceite

- definição inválida é rejeitada antes da execução;
- timeout e cancelamento são observáveis;
- stdout e stderr são separados e limitados;
- em execução assíncrona CTR-0005, stdout é protocolo estruturado consumido linha a linha e não é acumulado como saída textual; stderr permanece diagnóstico limitado;
- exit code é preservado quando disponível;
- exit code diferente de zero não é convertido automaticamente em falha de infraestrutura;
- o mesmo script pode possuir execuções simultâneas;
- grupo de processos de uma execução é encerrado no timeout/cancelamento;
- ambiente e working directory seguem CTR-0002.

## Implementação relacionada

Prevista para os pacotes internos de scripts e executor.
