# FLW-002 — Job assíncrono

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Objetivo

executar trabalho demorado sem bloquear o tratamento de novos comandos.  
## Ator inicial

ACT-001  
## Pré-condições

comando autorizado e operação classificada para processamento assíncrono.  
## Integrações

INT-001, INT-002, INT-004  
## Capacidades

IN-004, IN-006

## Sequência

```mermaid
sequenceDiagram
    actor U as Usuário
    participant T as Telegram
    participant S as Sabiá
    participant P as PostgreSQL
    participant W as Worker / operação

    U->>T: Solicita operação demorada
    T-->>S: Update
    S->>P: Cria requisição e job
    S->>T: Informa aceite do job
    S->>W: Worker assume job
    W-->>S: Progresso
    S->>P: Atualiza estado
    S->>T: Atualiza progresso
    W-->>S: Resultado final ou falha
    S->>P: Persiste estado terminal
    S->>T: Envia resultado
    T-->>U: Apresenta resultado
```

## Resultado esperado

O job evolui por estados controlados, pode informar progresso e termina com resultado ou falha sem impedir o processamento de outros comandos.

## Falhas relevantes

- fila indisponível;
- timeout ou falha do processador;
- interrupção do serviço;
- falha na entrega da atualização ou resultado.
