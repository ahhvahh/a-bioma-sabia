<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ARQ-004 — Registro e execução de scripts

**ID:** `ARQ-004` · **Origem:** `MOD-0003` · **Estado histórico:** `refined`  
**Implementação:** prevista, sem evidência de código na branch

## Objetivo

Resolver operações locais por identificador permitido e executá-las de forma controlada conforme CTR-0002.

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

## Entradas e saídas

**Entrada:** Identificador de script já autorizado e argumentos permitidos pela operação.

Caminho do executável, interpretador, diretório de trabalho e política de ambiente são resolvidos a partir da configuração e não da solicitação remota.

**Saída:** Resultado de execução conforme [CTR-0002 — Execução de script](../especificacao/contratos/execucao-script.md).

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

## Dependências e contratos

- [DSG-0002 — Componentes do Sabiá Core](ARQ-002-componentes-core.md)
- [ADR-0004 — Registro explícito de scripts](../decisoes/DEC-004-registro-explicito-scripts.md)
- [CTR-0002 — Execução de script](../especificacao/contratos/execucao-script.md)
- [CFG-0001 — Modelo de configuração](../especificacao/configuracao/modelo-configuracao.md)

## Implementação prevista

Prevista para os pacotes internos de scripts e executor. **Não verificada no código-fonte desta branch.**

## Pendências de integração

A origem estabelece Script Registry configurado previamente. Existe proposta posterior de catálogo persistente de scripts; a precedência entre YAML e PostgreSQL permanece aberta na [documentação de persistência](../persistencia/pendencias.md).

## Rastreabilidade

[MOD-0003 — especificação original](../especificacao/modulos/scripts.md). O identificador original permanece válido para documentos históricos.
