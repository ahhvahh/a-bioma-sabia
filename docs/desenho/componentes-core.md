# Componentes do Sabiá Core

![DSG](https://img.shields.io/badge/DSG-DSG--0002-0550ae?style=flat-square)
![Status](https://img.shields.io/badge/Status-finalized-0a7ea4?style=flat-square)

## Objetivo

Representar os componentes necessários ao primeiro MVP e suas responsabilidades principais.

## Dependências

- [ADR-0001 — Go, binário único e serviço Linux](../adr/runtime/go-binario-unico.md)
- [ADR-0002 — Múltiplos clientes Telegram isolados](../adr/telegram/multiplos-clientes-isolados.md)
- [ADR-0003 — Core independente do Telegram](../adr/arquitetura/core-independente-do-telegram.md)
- [ADR-0004 — Registro explícito de scripts](../adr/execucao/registro-explicito-de-scripts.md)
- [ADR-0005 — Telegram Long Polling](../adr/telegram/long-polling.md)
- [ADR-0006 — Jobs assíncronos](../adr/processamento/jobs-assincronos.md)
- [ADR-0007 — Scheduler e alertas orientados a estado](../adr/monitoramento/scheduler-alertas-estado.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../adr/seguranca/menor-privilegio-e-autorizacao.md)
- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](../adr/processamento/processadores-assincronos-registrados.md)

## Nível C4

`Component`

## Diagrama

```text
Telegram Bot API
      |
      v
Telegram Clients/Adapter
      |
      v
Authorization
      |
      v
Command Router
  |        |
  |        +--------------------+
  v                             v
Processor Registry          Job Manager
  |                            |
  v                            v
Processor Transport       Job Queue/Workers
  |                            |
  +----> Script/Executable <---+
  +----> Socket Service <------+

Scheduler ---> Processor Registry/Transport ---> Alert Manager ---> Telegram Adapter

Config ------------------------> todos os componentes
Logging/Audit <----------------- eventos operacionais
```

## Elementos e responsabilidades

- **Telegram Clients/Adapter:** long polling, tradução de updates e envio/edição de mensagens.
- **Authorization:** valida usuário e, quando configurado, chat.
- **Command Router:** resolve identificadores de comandos internos.
- **Processor Registry:** associa identificadores permitidos às definições de processadores cadastrados, incluindo scripts, aplicações e serviços/socket.
- **Processor Transport:** adapta cada mecanismo concreto de execução/comunicação ao protocolo interno de requisição, progresso e finalização; CTR-0002 continua sendo usado na execução local controlada.
- **Job Manager/Queue/Workers:** executa operações demoradas fora do tratamento imediato do comando, preserva `request_id` e encaminha eventos de progresso/finalização.
- **Scheduler:** dispara verificações cadastradas.
- **Alert Manager:** avalia mudança de estado e decide quando notificar.
- **Config:** carrega configuração sem tokens reais versionados.
- **Logging/Audit:** produz registros estruturados sem segredos.

## Relações relevantes

O Command Router não conhece detalhes de Telegram, shell, socket, FFmpeg ou outra ferramenta concreta. Processadores entram pelo Processor Registry e são acessados pela camada Processor Transport.

## Critérios para finalização

- responsabilidades do MVP estão separadas;
- componentes de transporte, domínio operacional e execução não estão fundidos;
- o Processor Transport separa protocolo interno de mecanismos concretos de execução/comunicação;
- detalhes ainda não refinados de framing, binário e persistência não alteram este desenho estrutural.
