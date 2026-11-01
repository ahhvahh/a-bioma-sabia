# FLW-003 — Monitoramento agendado e alerta

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Objetivo

executar uma verificação cadastrada e notificar somente quando a política de estado exigir.  
## Ator inicial

Scheduler interno  
## Pré-condições

schedule e operação de verificação cadastrados.  
## Integrações

INT-001, INT-002, INT-004  
## Capacidades

IN-005, IN-006

## Sequência

```mermaid
sequenceDiagram
    participant S as Scheduler / Sabiá
    participant O as Verificação cadastrada
    participant P as PostgreSQL
    participant T as Telegram
    actor U as Usuário autorizado

    S->>O: Executa verificação
    O-->>S: Estado atual
    S->>P: Consulta estado anterior
    S->>S: Avalia transição
    S->>P: Persiste novo estado
    alt Alerta necessário
        S->>T: Envia alerta ou recuperação
        T-->>S: Confirma aceitação
        T-->>U: Apresenta notificação
    else Sem alteração relevante
        S->>S: Não envia alerta imediato
    end
```

## Resultado esperado

Mudanças relevantes, recuperações e lembretes configurados podem gerar notificação; repetição de um mesmo estado não produz alerta imediato por padrão.

## Falhas relevantes

- verificação retorna estado desconhecido ou falha;
- Telegram indisponível;
- estado anterior não pode ser recuperado.
