# FLW-001 — Comando Telegram

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Objetivo

processar um comando autorizado e devolver sua resposta pelo mesmo contexto de cliente.  
## Ator inicial

ACT-001  
## Pré-condições

cliente habilitado; credencial disponível; usuário autorizado.  
## Integrações

INT-001, INT-002, INT-004  
## Capacidades

IN-001, IN-002, IN-003, IN-006

## Sequência

```mermaid
sequenceDiagram
    actor U as Usuário
    participant T as Telegram
    participant S as Sabiá
    participant P as PostgreSQL
    participant O as Operação cadastrada

    U->>T: Envia comando
    S->>T: Busca update por long polling
    T-->>S: Update
    S->>P: Persiste update e requisição
    S->>S: Autoriza e resolve operação
    S->>O: Executa operação cadastrada
    O-->>S: Resultado
    S->>P: Persiste resposta pendente
    S->>T: Envia resposta
    T-->>S: Confirma aceitação
    S->>P: Registra entrega
    T-->>U: Apresenta resposta
```

## Resultado esperado

A requisição é processada uma única vez do ponto de vista lógico e a resposta é enviada pelo cliente e destino correlacionados.

## Falhas relevantes

- usuário ou chat não autorizado;
- comando inexistente ou argumentos inválidos;
- falha temporária ou permanente da Telegram Bot API;
- falha da operação cadastrada;
- indisponibilidade da persistência necessária ao processamento.
