# Persistência do estado operacional

![ADR](https://img.shields.io/badge/ADR-ADR--0009-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

## Contexto

Jobs, alertas orientados a estado e long polling possuem informações operacionais que podem precisar sobreviver a reinicializações.

## Problema

Definir quais estados são persistentes e qual mecanismo mantém esses dados.

## Restrições

- jobs precisam ser correlacionados com chat e mensagem;
- alertas dependem do estado anterior;
- graceful shutdown prevê salvar estado necessário;
- duplicação de updates Telegram deve ser evitada;
- a solução deve permanecer simples e de baixo consumo.

## Opções consideradas

### Estado apenas em memória

Mais simples, mas perde contexto em reinício.

### Estado persistente local

Pode preservar jobs, offsets e estados de monitoramento, mas exige tecnologia, modelo e política de recuperação.

## Decisão

**BLOCKED.** O material de origem não define o mecanismo de persistência nem quais estados devem sobreviver ao restart.

## Justificativa

Escolher armazenamento ou política de recovery sem decisão do projeto criaria arquitetura não autorizada.

## Consequências

Enquanto este ADR estiver em `refinement`, ficam bloqueadas as partes das especificações que dependem de recovery de jobs, continuidade de alertas e confirmação persistente de updates.

## Dependências

- [ADR-0005 — Telegram Long Polling](../telegram/long-polling.md)
- [ADR-0006 — Jobs assíncronos](../processamento/jobs-assincronos.md)
- [ADR-0007 — Scheduler e alertas orientados a estado](../monitoramento/scheduler-alertas-estado.md)

## Critérios de validação

Para chegar a `refined`, definir:
- mecanismo de armazenamento;
- dados persistidos;
- comportamento de recovery;
- política de retenção/limpeza;
- tratamento de jobs interrompidos;
- persistência ou não do offset/identificador de update do Telegram.
