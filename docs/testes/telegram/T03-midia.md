<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# T03 — Limites e apresentação de mídia

**Natureza:** especificação de cenários de teste, não suíte executável. **Estado:** planejado; nenhuma evidência de execução, fixture implementada ou resultado PASS/FAIL nesta branch.

## Objetivo

Verificar limites em bytes decimais na ingestão local e seleção de métodos de apresentação Telegram, respeitando requisitos de segurança.

## Pré-condições previstas

- Bot API HTTP fake, sem token ou requisição ao Telegram real.
- PostgreSQL temporário/descartável com reset de estado por caso.
- Injeção de falhas conforme cenários; os mecanismos concretos de execução ainda não existem.

## Cenários a implementar

| Cenário | Estímulo/condição | Resultado esperado segundo a documentação |
|---|---|---|
| Upload integral no limite | Receber mídia local de até `20000000` bytes | Aceitar se demais validações passarem; ACK apenas após persistência confirmada |
| Upload integral acima do limite | Receber `20000001` bytes | Rejeitar com `media_too_large` sem persistir mídia |
| Abertura fracionada válida | Abrir com `total_bytes` entre 1 e `50000000` | Criar mídia incompleta e retornar `media_id` e `next_sequence_id = 1` após commit |
| Abertura fracionada excedida | Abrir com `total_bytes > 50000000` | Rejeitar antes de criar `media_id` |
| Chunk grande ou fora da sequência | Enviar chunk de mais de `5000000` bytes ou `sequence_id` inválido | Rejeitar conforme contrato; não avançar contadores em erro |
| Entrega de mídia fracionada | Solicitar transmissão antes/depois da conclusão | Não transmitir antes de `completed = true`; recompor stream na ordem de `sequence_id` |
| Foto JPEG/PNG compatível | Selecionar apresentação com MIME confirmado | Usar `sendPhoto` respeitando o limite específico da Bot API |
| Vídeo MP4 compatível | Selecionar apresentação com MIME confirmado | Usar `sendVideo` respeitando limite da API |
| Outro MIME ou desconhecido | Enviar mídia válida sem classificação segura como foto/vídeo | Usar `sendDocument` quando aceito; não contornar limites de método |
| Download público acima do limite | Tentar baixar entrada maior que `20000000` bytes via `getFile` público | Produzir falha controlada, não prometer download com Bot API pública |

## Evidências exigidas quando implementado

- Identificar caso, entrada fake, estado PostgreSQL anterior e posterior e chamadas HTTP capturadas.
- Registrar resultado PASS/FAIL **apenas após execução real**.
- Não usar credenciais reais, dados pessoais ou binários sensíveis nas fixtures versionadas.
- Indicar claramente dependências e comportamentos ainda em `refinement`.

## Fontes e rastreabilidade

- [Índice T01–T05](README.md)
- [REF-003 — Requisitos gerais de testes](../../referencia/REF-003-requisitos-testes.md)
- [CONTR-008 — Ingestão simples](../../contratos/CONTR-008-ingestao-midia-messagepack.md)
- [CONTR-009 — Ingestão fracionada](../../contratos/CONTR-009-ingestao-midia-fracionada.md)
- [REF-004 — Limites e apresentação Telegram](../../referencia/REF-004-operacao-telegram.md)
- [CONTR-007 — Transmissão](../../contratos/CONTR-007-transmissao-midia.md)

Este documento detalha critérios de aceite já especificados; não atesta implementação, aprovação de novos comportamentos ou integração real com Telegram.
