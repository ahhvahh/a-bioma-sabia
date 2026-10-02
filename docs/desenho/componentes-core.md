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
Script Registry/Executor     Job Manager
                               |
                               v
                         Job Queue/Workers

Scheduler ---> Script Executor ---> Alert Manager ---> Telegram Adapter

Config ------------------------> todos os componentes
Logging/Audit <----------------- eventos operacionais
```

## Elementos e responsabilidades

- **Telegram Clients/Adapter:** long polling, tradução de updates e envio/edição de mensagens.
- **Authorization:** valida usuário e, quando configurado, chat.
- **Command Router:** resolve identificadores de comandos internos.
- **Script Registry:** associa identificadores permitidos às definições de execução.
- **Script Executor:** executa definição já cadastrada e coleta resultado.
- **Job Manager/Queue/Workers:** executa operações demoradas fora do tratamento imediato do comando.
- **Scheduler:** dispara verificações cadastradas.
- **Alert Manager:** avalia mudança de estado e decide quando notificar.
- **Config:** carrega configuração sem tokens reais versionados.
- **Logging/Audit:** produz registros estruturados sem segredos.

## Relações relevantes

O Command Router não conhece detalhes de Telegram, shell, FFmpeg ou outra ferramenta concreta. Ferramentas futuras entram por executores.

## Critérios para finalização

- responsabilidades do MVP estão separadas;
- componentes de transporte, domínio operacional e execução não estão fundidos;
- pontos ainda não decididos de persistência não alteram este desenho estrutural.
