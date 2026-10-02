# Processadores assíncronos registrados e transporte de progresso

![ADR](https://img.shields.io/badge/ADR-ADR--0011-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

O Sabiá precisa delegar trabalhos a scripts Bash, aplicações locais e serviços acessíveis por socket. Parte desses trabalhos é assíncrona e pode produzir mensagens de progresso e arquivos de resultado, incluindo imagens e vídeos destinados ao cliente Telegram.

## Problema

O modelo atual de execução de script é orientado à conclusão do processo e captura de stdout/stderr. Isso não é suficiente para processadores que continuam trabalhando e precisam publicar progresso até uma mensagem explícita de finalização.

Também é necessário preservar a independência entre Core, Telegram e tecnologia concreta do processador.

## Restrições

- processadores precisam ser previamente cadastrados;
- comandos remotos não escolhem caminho arbitrário, executável ou socket;
- o Sabiá gera e controla o identificador da requisição;
- o processador precisa devolver esse identificador em toda mensagem de progresso ou finalização;
- o Core não deve depender de objetos da Telegram Bot API;
- arquivos de resultado podem incluir imagem, vídeo ou outro arquivo suportado pelo transporte;
- contratos binários e limites precisam ser definidos antes da implementação.

## Opções consideradas

### Apenas execução local orientada a exit code

Preserva CTR-0002, mas não atende aplicações e serviços assíncronos que publicam progresso antes da conclusão.

### Processadores registrados com protocolo assíncrono comum

Mantém diferentes tecnologias de execução atrás de uma fronteira única e permite progresso e resultado final correlacionados.

## Decisão

O Sabiá terá uma abstração de **processador registrado** capaz de representar:

- script Bash;
- aplicação/executável;
- serviço acessível por socket.

A comunicação com processadores assíncronos será mediada por uma camada de **Processor Transport**, responsável por adaptar o mecanismo concreto de cada processador ao protocolo interno comum.

Cada execução recebe um `request_id` gerado pelo Sabiá.

Durante a execução, o processador envia eventos correlacionados contendo:

- `request_id`;
- `status`, com valores `loading`, `content` ou `finally`;
- `message`.

`loading` representa progresso intermediário e pode ser encaminhado ao cliente de origem.

`content` anuncia um arquivo produzido dentro da área temporária da requisição e pode ocorrer múltiplas vezes.

`finally` encerra semanticamente o processamento daquela requisição e carrega a mensagem final. Arquivos não são transportados dentro de `finally`; a referência e o streaming seguem ADR-0012.

CTR-0002 continua válido para a execução local controlada de scripts/executáveis. O novo protocolo assíncrono complementa CTR-0002 e não transforma stdout/stderr em contrato de progresso.

## Justificativa

A abstração permite usar scripts, aplicações e serviços sem acoplar o Command Router ou o Telegram à forma concreta de execução e sem confundir término do processo com conclusão semântica do trabalho assíncrono.

## Consequências

- será necessário um registro de processadores e seus tipos de transporte;
- jobs precisam preservar `request_id` até a finalização e entrega;
- mensagens `loading` precisam ser encaminhadas pelo contexto de resposta persistido;
- o evento `finally` precisa produzir resultado persistente antes da entrega;
- o contrato de referência/lifecycle da mídia e o framing dos transportes concretos ainda precisam ser refinados;
- saída de processo e protocolo assíncrono passam a ser conceitos distintos.

## Dependências

- [ADR-0003 — Core independente do Telegram](../arquitetura/core-independente-do-telegram.md)
- [ADR-0004 — Registro explícito de scripts](../execucao/registro-explicito-de-scripts.md)
- [ADR-0006 — Jobs assíncronos](jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../persistencia/estado-operacional.md)
- [ADR-0012 — Área temporária de mídia para processadores](midia-temporaria-por-request.md)

## Critérios de validação

- script, aplicação e serviço/socket podem ser representados por uma operação cadastrada sem fornecer endereço arbitrário pelo Telegram;
- toda mensagem do processador contém o `request_id` fornecido pelo Sabiá;
- `loading` pode gerar atualização ao cliente sem finalizar o job;
- `content` pode publicar vários arquivos sem finalizar o job;
- `finally` encerra semanticamente a requisição;
- binários não trafegam no protocolo de controle;
- detalhes de framing e lifecycle da mídia permanecem em especificação, não no Command Router.
