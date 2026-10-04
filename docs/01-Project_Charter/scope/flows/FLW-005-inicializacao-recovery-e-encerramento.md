# FLW-005 — Inicialização, recovery e encerramento

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Objetivo

iniciar e encerrar o serviço preservando consistência do trabalho operacional.  
## Ator inicial

ACT-002 / systemd  
## Pré-condições

instalação e configuração operacional disponíveis.  
## Integrações

INT-004, INT-005, INT-001  
## Capacidades

IN-006, IN-009

## Sequência

```mermaid
sequenceDiagram
    actor O as Operador
    participant D as systemd / Linux
    participant S as Sabiá
    participant P as PostgreSQL
    participant T as Telegram

    O->>D: Inicia ou reinicia serviço
    D->>S: Start
    S->>P: Carrega estado recuperável
    S->>S: Reconcilia trabalho pendente
    S->>T: Retoma entregas quando aplicável
    O->>D: Stop / restart
    D->>S: SIGTERM ou SIGINT
    S->>S: Interrompe aceitação de novo trabalho
    S->>P: Preserva estado necessário
    S-->>D: Encerra de forma controlada
```

## Resultado esperado

O processo inicia com recuperação do estado relevante e encerra sem depender de execução como root ou perda silenciosa de trabalho persistente.

## Falhas relevantes

- PostgreSQL indisponível durante startup;
- configuração inválida;
- trabalho interrompido sem estado recuperável;
- falha de comunicação externa durante a retomada.
