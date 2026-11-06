<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# T02 — Menus, callbacks e autorização

**Natureza:** especificação de cenários de teste, não suíte executável. **Estado:** planejado; nenhuma evidência de execução, fixture implementada ou resultado PASS/FAIL nesta branch.

## Objetivo

Verificar navegação Telegram, revalidação de permissões e não execução de ações por cliques indevidos.

## Pré-condições previstas

- Bot API HTTP fake, sem token ou requisição ao Telegram real.
- PostgreSQL temporário/descartável com reset de estado por caso.
- Injeção de falhas conforme cenários; os mecanismos concretos de execução ainda não existem.

## Cenários a implementar

| Cenário | Estímulo/condição | Resultado esperado segundo a documentação |
|---|---|---|
| `/start` / `/menu` autorizado | Enviar comando de usuário autorizado | Gerar `InlineKeyboardMarkup` somente das ações permitidas |
| Contato não autorizado | Enviar `/start` e tentar ação restrita | Registrar contato sem conceder autorização; negar execução |
| Categoria | Clicar botão de categoria | Receber `callback_query`, responder via `answerCallbackQuery` e trocar menu com `editMessageText` quando possível |
| Falha de edição do menu | Simular mensagem não editável | Enviar novo menu segundo a regra documental |
| Voltar e Início | Selecionar ambas as ações | Voltar à categoria-pai ou à raiz, respectivamente |
| Callback obsoleto ou forjado | Injetar callback sem autorização/fora do estado atual | Revalidar bot, usuário, chat e ação; recusar execução indevida |
| Limite e confidencialidade | Fornecer `callback_data` e inspecionar conteúdo | Usar identificador compacto de até 64 bytes, sem caminhos, parâmetros secretos ou autorização implícita |

## Evidências exigidas quando implementado

- Identificar caso, entrada fake, estado PostgreSQL anterior e posterior e chamadas HTTP capturadas.
- Registrar resultado PASS/FAIL **apenas após execução real**.
- Não usar credenciais reais, dados pessoais ou binários sensíveis nas fixtures versionadas.
- Indicar claramente dependências e comportamentos ainda em `refinement`.

## Fontes e rastreabilidade

- [Índice T01–T05](README.md)
- [REF-003 — Requisitos gerais de testes](../../referencia/REF-003-requisitos-testes.md)
- [REF-004 — Navegação Telegram](../../referencia/REF-004-operacao-telegram.md)
- [FLUX-001 — Callback e autorização](../../fluxos/FLUX-001-comando-telegram.md)
- [ARQ-006 — Segurança](../../arquitetura/ARQ-006-seguranca-autorizacao.md)

Este documento detalha critérios de aceite já especificados; não atesta implementação, aprovação de novos comportamentos ou integração real com Telegram.
