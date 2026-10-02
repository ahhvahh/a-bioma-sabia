# Modelo de configuração

**ID:** CFG-0001  
**Status:** refinement

## Objetivo

Registrar a configuração conhecida do Sabiá e as lacunas que ainda impedem um schema completo.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](../../adr/telegram/multiplos-clientes-isolados.md)
- [ADR-0004 — Registro explícito de scripts](../../adr/execucao/registro-explicito-de-scripts.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../../adr/seguranca/menor-privilegio-e-autorizacao.md)
- [MOD-0002 — Adaptador Telegram](../modulos/telegram.md)
- [MOD-0003 — Registro e execução de scripts](../modulos/scripts.md)
- [MOD-0005 — Scheduler e Alert Manager](../modulos/scheduler-alertas.md)

## Fontes previstas

O projeto prevê configuração YAML. Tokens reais não são gravados nesses arquivos; clientes referenciam variável de ambiente ou outra fonte protegida.

Arquivos conceituais previstos:
- `sabia.yaml`;
- `commands.yaml`;
- `schedules.yaml`.

A separação física final ainda não é normativa.

## Clientes Telegram

Cada cliente precisa suportar:
- `enabled`;
- referência do token, por exemplo `token_env`;
- `allowed_users`;
- `allowed_chats` opcional;
- catálogo próprio de comandos/scripts/permissões.

Clientes iniciais:
- `bioma`;
- `tools`;
- `alerts`.

## Registro de scripts

Cada item possui:
- identificador lógico;
- caminho local configurado;
- timeout.

Exemplos conceituais do escopo:
- `server-status` → script de status;
- `disk-check` → monitoramento de disco;
- `backup` → operação de backup.

## Scheduler

Cada tarefa precisa relacionar:
- identificador;
- operação/script cadastrado;
- periodicidade;
- política de relatório;
- configuração de lembrete quando aplicável.

Exemplos iniciais:
- disk-check: 5 minutos;
- service-check: 1 minuto;
- backup-check: 30 minutos;
- temperature: 10 minutos.

Esses exemplos não substituem a definição do schema.

## BLOCKED

Antes de `refined`, definir:
- schema completo e tipos;
- precedência entre arquivos/fontes;
- comportamento de valor ausente ou inválido;
- recarga dinâmica ou somente no startup;
- localização dos arquivos;
- regras exatas para fonte de segredo em arquivo protegido;
- schema de comandos e schedules.
