# Auditoria e logs

![CTR](https://img.shields.io/badge/CTR-CTR--0004-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Padronizar informações mínimas de observabilidade e auditoria sem registrar segredos.

## Dependências

- [MOD-0006 — Segurança e autorização](../modulos/seguranca.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../../adr/seguranca/menor-privilegio-e-autorizacao.md)

## Tipo

`evento`

## Entrada

Eventos operacionais do serviço.

## Saída

Logs estruturados devem poder representar:
- timestamp;
- level;
- component;
- client;
- command;
- job_id;
- message.

Auditoria de comandos deve poder representar:
- timestamp;
- client;
- telegram_user_id;
- chat_id;
- command;
- job_id;
- resultado;
- tempo.

Campos não aplicáveis podem estar ausentes, mas o significado não deve mudar.

## Erros

Falha de logging não autoriza revelar segredo nem transformar dado sensível em fallback textual.

## Regras e restrições

Nunca registrar:
- tokens;
- senhas;
- segredos;
- credenciais;
- BLOBs de mídia;
- campo `data` de CTR-0008.

A saída inicial é stdout/stderr para integração natural com systemd/journalctl.

## Compatibilidade

O formato deve permanecer estruturado mesmo quando novos componentes forem adicionados.

## BLOCKED

Antes de retornar a `refined`, definir:

- tipos dos campos estruturados;
- campos obrigatórios por família de evento;
- identificador/nome estável de cada evento auditável;
- resultado/estado permitido para cada família;
- tratamento de identificadores potencialmente sensíveis, além da proibição já definida de segredos e BLOBs.

## Critérios de aceite

- eventos incluem contexto suficiente para correlação;
- segredos são omitidos;
- conteúdo binário de mídia nunca é serializado para logs/auditoria;
- saída é consumível por journalctl através de stdout/stderr.
