# Registro explícito de scripts

![ADR](https://img.shields.io/badge/ADR-ADR--0004-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

O MVP usa scripts locais para consultar o sistema e executar verificações.

## Problema

Permitir automação sem transformar mensagens Telegram em comandos arbitrários do sistema operacional.

## Restrições

- caminhos arbitrários informados pelo usuário são proibidos;
- texto Telegram não pode ser concatenado em `bash -c`;
- cada script possui identificador cadastrado e timeout;
- o usuário remoto conhece apenas o identificador lógico permitido.

## Opções consideradas

### Execução livre de shell

Incompatível com o requisito de segurança.

### Registro explícito de scripts

Restringe execução a operações previamente aprovadas.

## Decisão

Toda execução de script originada pelo Sabiá deve resolver um identificador em um Script Registry previamente configurado. Nenhum caminho ou comando de shell arbitrário recebido do Telegram pode ser executado.

## Justificativa

Reduz a superfície de injeção e mantém a lista de operações controlável.

## Consequências

- novos scripts precisam ser cadastrados antes de uso;
- o executor recebe uma definição já resolvida pelo registro;
- o contrato precisa capturar stdout, stderr, exit code, duração e timeout.

## Dependências

- [ADR-0003 — Core independente do Telegram](../arquitetura/core-independente-do-telegram.md)

## Critérios de validação

- `/run server-status` pode resolver um script cadastrado;
- `/run /tmp/x.sh` não executa um caminho não cadastrado;
- mensagem Telegram nunca é usada como corpo de shell.
