<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# T04 — Retry, backoff, falha ambígua e reinício

**Natureza:** especificação de cenários de teste, não suíte executável. **Estado:** planejado; nenhuma evidência de execução, fixture implementada ou resultado PASS/FAIL nesta branch.

## Objetivo

Verificar tratamento de falhas temporárias e persistência da fila de respostas sem prometer entrega remota exatamente uma vez.

## Pré-condições previstas

- Bot API HTTP fake, sem token ou requisição ao Telegram real.
- PostgreSQL temporário/descartável com reset de estado por caso.
- Injeção de falhas conforme cenários; os mecanismos concretos de execução ainda não existem.

## Cenários a implementar

| Cenário | Estímulo/condição | Resultado esperado segundo a documentação |
|---|---|---|
| Falha transitória de leitura | Responder 5xx/timeout a `getUpdates` | Manter offset; backoff documentado de 1s, 2s, 4s, 8s, 16s até 30s |
| Leitura bem-sucedida após falhas | Retornar resposta válida de polling, mesmo sem updates | Reiniciar backoff em 1s |
| Rate limit 429 | Responder 429 com `retry_after` | Não tentar novamente antes do intervalo informado |
| 401 ou 403 na leitura | Simular autenticação/autorização negada | Suspender somente o cliente afetado, mantendo demais clientes |
| Falha temporária ou ambígua de envio | Desconectar ou perder confirmação ao enviar mensagem | Manter ou devolver mensagem a `pending`; tentativa pode produzir duplicidade remota |
| Reinício com mensagem `sending` | Reiniciar o serviço antes da confirmação local | Reconciliar `sending` para `pending` e permitir retry |
| Mensagem `delivered` | Reiniciar após confirmação remota persistida | Não reenviar entrega concluída |
| Falha permanente | Registrar falha irreversível do transporte | Marcar `failed` e não repetir automaticamente sem nova ação definida |

## Evidências exigidas quando implementado

- Identificar caso, entrada fake, estado PostgreSQL anterior e posterior e chamadas HTTP capturadas.
- Registrar resultado PASS/FAIL **apenas após execução real**.
- Não usar credenciais reais, dados pessoais ou binários sensíveis nas fixtures versionadas.
- Indicar claramente dependências e comportamentos ainda em `refinement`.

## Fontes e rastreabilidade

- [Índice T01–T05](README.md)
- [REF-003 — Requisitos gerais de testes](../../referencia/REF-003-requisitos-testes.md)
- [FLUX-001 — Backoff e estados](../../fluxos/FLUX-001-comando-telegram.md)
- [REF-004 — Entrega at-least-once](../../referencia/REF-004-operacao-telegram.md)
- [CONTR-007 — Recovery de mídia](../../contratos/CONTR-007-transmissao-midia.md)

Este documento detalha critérios de aceite já especificados; não atesta implementação, aprovação de novos comportamentos ou integração real com Telegram.
