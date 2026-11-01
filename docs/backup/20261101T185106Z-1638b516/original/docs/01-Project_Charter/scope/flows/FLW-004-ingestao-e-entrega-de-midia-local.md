# FLW-004 — Ingestão e entrega de mídia local

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Objetivo

receber mídia de produtor local autorizado, persistir o conteúdo e entregá-lo pelo contexto Telegram correlacionado.  
## Ator inicial

ACT-003  
## Pré-condições

produtor autorizado e requisição/destino correlacionável.  
## Integrações

INT-003, INT-004, INT-001  
## Capacidades

IN-006, IN-007, IN-008

## Sequência

```mermaid
sequenceDiagram
    actor A as Aplicação local
    participant S as Sabiá
    participant P as PostgreSQL
    participant T as Telegram
    actor U as Usuário

    A->>S: Abre ingestão com metadados
    S->>P: Cria registro de mídia
    P-->>S: media_id
    S-->>A: Confirma abertura
    A->>S: Envia conteúdo integral ou chunks
    S->>P: Persiste conteúdo progressivamente
    S-->>A: Confirma conteúdo aceito
    S->>P: Cria transmissão pendente
    S->>T: Envia arquivo lógico
    T-->>S: Confirma aceitação
    S->>P: Marca transmissão entregue
    T-->>U: Disponibiliza mídia
```

## Resultado esperado

Mídia aceita permanece persistida e pode ser transmitida ao destino correto, inclusive após restart enquanto ainda estiver pendente.

## Falhas relevantes

- produtor sem autorização local;
- payload acima do limite permitido no canal escolhido;
- sequência de chunks inválida;
- mídia incompleta;
- falha temporária ou permanente na transmissão externa.
