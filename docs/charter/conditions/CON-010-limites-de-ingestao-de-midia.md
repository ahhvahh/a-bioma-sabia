# CON-010 — Limites de ingestão de mídia

![Tipo: CON](../assets/documentation-style/badges/badge-type-con.svg) ![Status: refinement](../assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](../assets/documentation-style/badges/badge-version-v1.svg)

## Restrição

Ingestão simples: no máximo 20 MB. Por partes: chunks de até 5 MB e total de até **50.000.000 bytes (50 MB decimais)**. A decisão registrada em 21/10/2026 especifica a unidade decimal e prevalece sobre a redação histórica genérica. Limites de API externa podem ser inferiores e precisam ser verificados antes da transferência.

## Origem e motivação

Decisão documentada no Charter do Sabiá, preservada na migração. Limita as opções de implementação; não cria interfaces ou contratos técnicos.

## Verificação e impactos

- Verificar observância em integração e testes da etapa correspondente.
- PENDING — Scope/Implementation: contratos e testes de conformidade; sem links para documentos legados.
- Evidência de origem documental mantida no snapshot histórico; o snapshot não é referência ativa.
