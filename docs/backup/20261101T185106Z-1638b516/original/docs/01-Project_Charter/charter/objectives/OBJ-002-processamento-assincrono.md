# OBJ-002 — Manter operações demoradas fora do tratamento síncrono de comandos.

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Objetivo

Manter operações demoradas fora do tratamento síncrono de comandos.

## Motivação

Formalizar uma responsabilidade necessária ao propósito do Sabiá sem ampliar a fronteira além do Scope aprovado.

## Resultado esperado

Operações demoradas podem ser executadas como jobs sem impedir o recebimento de novos comandos e podem informar progresso e resultado.

## Critérios verificáveis

- Operações demoradas podem ser executadas como jobs sem impedir o recebimento de novos comandos e podem informar progresso e resultado.

## Limites

Este objetivo não autoriza responsabilidades além dos itens de escopo relacionados.

## Relações

- Scope: IN-004
- Fluxos: FLW-002
