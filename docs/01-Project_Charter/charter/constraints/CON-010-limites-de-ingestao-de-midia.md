# CON-010 — Limites de ingestão de mídia

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

**Disposition:** active

## Restrição

Ingestão simples de mídia aceita no máximo 20 MB; o canal fracionado usa chunks de até 5 MB e total de até 100 MB.

## Origem

Decisão vigente registrada no Scope anterior.

## Justificativa

Limitar explicitamente o volume aceito pelas interfaces de ingestão de mídia.

## Impacto

Arquivos acima do limite simples precisam usar ingestão fracionada e arquivos acima do total permitido são recusados.

## Evidências

- DOCUMENTED — Scope anterior do Sabiá.

## Itens relacionados

- IN-007
- FLW-004
