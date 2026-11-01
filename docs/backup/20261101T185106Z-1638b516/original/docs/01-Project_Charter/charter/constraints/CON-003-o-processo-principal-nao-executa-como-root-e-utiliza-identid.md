# CON-003 — O processo principal não executa como root e utiliza identidade Linux dedicada.

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Restrição

O processo principal não executa como root e utiliza identidade Linux dedicada.

## Origem

Requisito de segurança vigente.

## Justificativa

A restrição integra a baseline documental vigente e limita a realização das capacidades do Sabiá.

## Impacto

Operações que exijam privilégios adicionais precisam de mecanismo externo explicitamente controlado.

## Evidências

- DOCUMENTED — Project Charter e Scope anteriores do Sabiá.

## Itens relacionados

- Ver [Scope](../../scope/README.md).
