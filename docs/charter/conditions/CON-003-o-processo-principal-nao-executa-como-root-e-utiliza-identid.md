# CON-003 — Processo sem root

![Tipo: CON](../assets/documentation-style/badges/badge-type-con.svg) ![Status: refinement](../assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](../assets/documentation-style/badges/badge-version-v1.svg)

## Restrição

O processo principal usa identidade Linux dedicada e não executa como root.

## Origem e motivação

Decisão documentada no Charter do Sabiá, preservada na migração. Limita as opções de implementação; não cria interfaces ou contratos técnicos.

## Verificação e impactos

- Verificar observância em integração e testes da etapa correspondente.
- PENDING — Scope/Implementation: contratos e testes de conformidade; sem links para documentos legados.
- Evidência de origem documental mantida no snapshot histórico; o snapshot não é referência ativa.
