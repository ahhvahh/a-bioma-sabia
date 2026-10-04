# IN-013 — Assinaturas periódicas

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** backlog

## Capacidade

Permitir que um contato seja associado a uma ação periódica para receber conteúdo ou mensagens automáticas, incluindo cenários como `/assinaturas/notícias/naval`.

## Descrição verificável

A evolução futura deve reutilizar `contact` e `action` para associar destinatário, conteúdo e periodicidade sem exigir nova interação a cada entrega. Esta capacidade é aceita, mas não integra a execução corrente do primeiro MVP.

## Objetivo relacionado

Evolução futura registrada por NOBJ-006 e delimitada por OUT-009.

## Entradas

- Contato persistente.
- Ação persistente.
- Periodicidade e configuração a refinar futuramente.

## Saídas

- Entrega periódica correlacionada ao contato quando promovida para `active`.

## Fluxos relacionados

- Nenhum fluxo active nesta baseline.

## Integrações relacionadas

- INT-001
- INT-004
