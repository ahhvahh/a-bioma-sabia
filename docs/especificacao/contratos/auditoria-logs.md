# Auditoria e logs

![CTR](https://img.shields.io/badge/CTR-CTR--0004-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

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
- credenciais.

A saída inicial é stdout/stderr para integração natural com systemd/journalctl.

## Compatibilidade

O formato deve permanecer estruturado mesmo quando novos componentes forem adicionados.

## Critérios de aceite

- eventos incluem contexto suficiente para correlação;
- segredos são omitidos;
- saída é consumível por journalctl através de stdout/stderr.
