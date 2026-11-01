# FLW-007 — Cadastro administrativo e execução de ação persistente

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Objetivo

permitir que uma ação seja cadastrada pelo operador e posteriormente utilizada pelo usuário sem que o usuário remoto conheça ou escolha o caminho do script.  
## Ator inicial

ACT-002 e ACT-001  
## Pré-condições

operador com acesso administrativo local; cliente Telegram habilitado; ação válida e habilitada.  
## Integrações

INT-001, INT-002, INT-004  
## Capacidades

IN-003, IN-004, IN-006, IN-008, IN-010, IN-011, IN-012

## Sequência

```mermaid
sequenceDiagram
    actor O as Operador
    participant CLI as sabia CLI
    participant DB as PostgreSQL
    actor U as Usuário
    participant T as Telegram
    participant S as Sabiá
    participant P as Script / processador

    O->>CLI: Cadastra /acoes/videoytb + script
    CLI->>DB: Persiste ação

    U->>T: /start
    T->>S: Update
    S->>DB: Registra/atualiza contato
    S->>DB: Consulta ações habilitadas
    S-->>T: Apresenta menu

    U->>T: Escolhe ação e informa URL
    T->>S: Seleção + argumento
    S->>DB: Resolve ação persistida
    S->>DB: Cria correlação/job quando necessário
    S->>P: Executa script cadastrado
    P-->>S: Progresso / finalização
    P->>S: Publica arquivo correlacionado
    S->>DB: Persiste resultado e entrega
    S-->>T: Envia arquivo
    T-->>U: Arquivo disponível
```

## Resultado esperado

O operador consegue ampliar o catálogo sem recompilar o Sabiá. O usuário aciona a ação pelo menu, fornece somente os argumentos permitidos e recebe progresso, mensagem ou arquivo produzido pela operação.

## Falhas relevantes

- ação inexistente, desabilitada ou não autorizada;
- definição administrativa inválida;
- script/processador inexistente ou indisponível;
- argumento inválido, como URL fora do domínio aceito;
- timeout ou falha do processamento;
- falha na publicação ou entrega do arquivo.
