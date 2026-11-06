<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# T05 — Isolamento entre clientes Telegram

**Natureza:** especificação de cenários de teste, não suíte executável. **Estado:** planejado; nenhuma evidência de execução, fixture implementada ou resultado PASS/FAIL nesta branch.

## Objetivo

Verificar que `bioma`, `tools` e `alerts` não compartilham permissões, cursores, credenciais ou operações por acidente.

## Pré-condições previstas

- Bot API HTTP fake, sem token ou requisição ao Telegram real.
- PostgreSQL temporário/descartável com reset de estado por caso.
- Injeção de falhas conforme cenários; os mecanismos concretos de execução ainda não existem.

## Cenários a implementar

| Cenário | Estímulo/condição | Resultado esperado segundo a documentação |
|---|---|---|
| Três clientes lógicos | Inicializar `bioma`, `tools` e `alerts` com credenciais fake independentes | Associar cada sessão e update ao `client_id` correto |
| IDs iguais entre clientes | Enviar mesmo `update_id` para dois clientes distintos | Respeitar chave lógica por `(client_id, transport, update_id)`; não deduplicar globalmente |
| Lista de autorização distinta | Autorizar identidade em um cliente e não em outro | Permitir apenas ação do cliente explicitamente autorizado |
| Erro de token em um cliente | Produzir 401/403 no mock de somente um bot | Suspender somente seu polling; outros clientes permanecem elegíveis |
| Falha/backoff isolados | Injetar 5xx/429 em um cliente | Não transferir seu offset, temporização ou erro aos outros |
| Menu e ação exclusivos | Apresentar catálogo de ações por cliente | Não disponibilizar a um bot ações de outro sem vínculo e autorização expressos |

## Evidências exigidas quando implementado

- Identificar caso, entrada fake, estado PostgreSQL anterior e posterior e chamadas HTTP capturadas.
- Registrar resultado PASS/FAIL **apenas após execução real**.
- Não usar credenciais reais, dados pessoais ou binários sensíveis nas fixtures versionadas.
- Indicar claramente dependências e comportamentos ainda em `refinement`.

## Fontes e rastreabilidade

- [Índice T01–T05](README.md)
- [REF-003 — Requisitos gerais de testes](../../referencia/REF-003-requisitos-testes.md)
- [DEC-002 — Clientes isolados](../../decisoes/DEC-002-clientes-telegram-isolados.md)
- [FLUX-001 — Contexto e autorização](../../fluxos/FLUX-001-comando-telegram.md)
- [REF-004 — Inbox e callbacks](../../referencia/REF-004-operacao-telegram.md)
- [REF-001 — Configuração dos clientes](../../referencia/REF-001-modelo-configuracao.md)

Este documento detalha critérios de aceite já especificados; não atesta implementação, aprovação de novos comportamentos ou integração real com Telegram.
