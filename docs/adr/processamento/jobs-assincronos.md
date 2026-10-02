# Jobs assíncronos

![ADR](https://img.shields.io/badge/ADR-ADR--0006-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

Downloads, conversões e outras operações demoradas não podem bloquear o recebimento de novos comandos.

## Problema

Definir como o Sabiá executa trabalho demorado.

## Restrições

- resposta ao usuário deve ser rápida;
- progresso deve poder atualizar a mensagem do job;
- o resultado final pode incluir arquivo;
- são necessários JobManager, JobQueue e Worker.

## Opções consideradas

### Execução síncrona no tratamento do comando

Bloqueia a capacidade de receber novos comandos.

### Job assíncrono

Separa aceitação do comando e execução demorada.

## Decisão

Operações demoradas serão representadas por jobs assíncronos processados por fila e workers.

Estados previstos: `queued`, `running`, `completed`, `failed`, `cancelled` e `timeout`.

## Justificativa

Mantém o canal responsivo e permite observabilidade de progresso.

## Consequências

- o sistema precisa correlacionar job e destino de resposta;
- chat, mensagem e job precisam ser correlacionáveis para atualização de progresso;
- persistência, concorrência, retries e recovery ainda precisam ser fechados em especificação.

## Dependências

- [ADR-0003 — Core independente do Telegram](../arquitetura/core-independente-do-telegram.md)

## Critérios de validação

- um job demorado não bloqueia o processamento de outro comando;
- o estado do job evolui por estados válidos;
- progresso pode ser refletido na mensagem associada.
