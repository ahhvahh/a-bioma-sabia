# FLW-006 — /start, registro de contato e apresentação do menu

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Objetivo

registrar ou atualizar o contato que iniciou interação com o cliente Telegram e apresentar as ações disponíveis para aquele contexto.  
## Ator inicial

ACT-001  
## Pré-condições

cliente Telegram habilitado.  
## Integrações

INT-001, INT-004  
## Capacidades

IN-001, IN-002, IN-006, IN-010, IN-011

## Sequência

```mermaid
sequenceDiagram
    actor U as Usuário
    participant T as Telegram
    participant S as Sabiá
    participant P as PostgreSQL

    U->>T: /start
    S->>T: Busca update
    T-->>S: Update com usuário e chat
    S->>P: Registra/atualiza contact
    S->>P: Consulta ações habilitadas
    P-->>S: Catálogo aplicável
    S->>S: Aplica autorização e monta menu
    S->>T: Envia menu inicial
    T-->>U: Apresenta ações disponíveis
```

## Resultado esperado

O contato fica persistido e o usuário recebe uma representação do menu baseada no catálogo de ações. O cadastro do contato não amplia automaticamente suas permissões.

## Falhas relevantes

- dados mínimos de identidade/destino ausentes;
- persistência indisponível;
- falha no envio do menu;
- contato conhecido, porém sem autorização para ações restritas.
