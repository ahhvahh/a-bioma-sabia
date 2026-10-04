# IN-010 — Registro persistente de contatos

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Capacidade

Registro persistente de contatos

## Descrição verificável

Criar ou atualizar o contato relacionado ao cliente Telegram, usuário e destino quando ocorrer `/start`, preservando datas de primeiro e último contato e estado necessário para comunicação posterior.

## Objetivo relacionado

OBJ-005, OBJ-007

## Entradas

- /start com identidade, cliente e destino.

## Saídas

- Contato criado ou atualizado sem concessão automática de permissão.

## Fluxos relacionados

- FLW-006
- FLW-007

## Integrações relacionadas

- INT-001
- INT-004
