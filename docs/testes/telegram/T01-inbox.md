<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# T01 — Inbox, cursor, crash e replay

**Natureza:** especificação de cenários de teste, não suíte executável. **Estado:** planejado; nenhuma evidência de execução, fixture implementada ou resultado PASS/FAIL nesta branch.

## Objetivo

Verificar a durabilidade do `Update` integral e a correlação do cursor com o commit local, evitando repetição lógica de processamento.

## Pré-condições previstas

- Bot API HTTP fake, sem token ou requisição ao Telegram real.
- PostgreSQL temporário/descartável com reset de estado por caso.
- Injeção de falhas conforme cenários; os mecanismos concretos de execução ainda não existem.

## Cenários a implementar

| Cenário | Estímulo/condição | Resultado esperado segundo a documentação |
|---|---|---|
| Aceite atômico | Receber update válido com `client_id`, `transport` e `update_id` | Gravar `raw_update` e avançar `transport_cursor` na mesma transação; somente após commit liberar offset maior |
| Falha antes do `INSERT` | Injetar falha antes da escrita | Não confirmar update nem avançar offset; próxima leitura pode recuperar o update |
| Falha entre `INSERT` e commit | Abortar transação após inserir inbox e antes do commit | Rollback de inbox/cursor; sem confirmação ou offset avançado |
| Crash após commit e antes do próximo `getUpdates` | Reiniciar após persistência confirmada | Update permanece recuperável no PostgreSQL, sem execução lógica duplicada |
| Replay do mesmo `update_id` | Repetir update para o mesmo `client_id`/transporte | Chave UNIQUE impede nova entrada lógica e segunda execução |
| Crash após claim ou durante script | Interromper consumo em `processing` | Reconciliar estado conforme recuperação definida; não executar novamente operação já concluída |
| Falha após envio e antes de gravar `message_id` | Perder confirmação local com envio remoto potencialmente efetivado | Registrar risco de duplicidade de resposta sob `at-least-once`; não prometer exatamente uma entrega |

## Evidências exigidas quando implementado

- Identificar caso, entrada fake, estado PostgreSQL anterior e posterior e chamadas HTTP capturadas.
- Registrar resultado PASS/FAIL **apenas após execução real**.
- Não usar credenciais reais, dados pessoais ou binários sensíveis nas fixtures versionadas.
- Indicar claramente dependências e comportamentos ainda em `refinement`.

## Fontes e rastreabilidade

- [Índice T01–T05](README.md)
- [REF-003 — Requisitos gerais de testes](../../referencia/REF-003-requisitos-testes.md)
- [REF-004 — Inbox e crash tests](../../referencia/REF-004-operacao-telegram.md)
- [FLUX-001 — Comando Telegram](../../fluxos/FLUX-001-comando-telegram.md)
- [ENT-011+ — Modelo de entidades](../../persistencia/entidades/README.md)

Este documento detalha critérios de aceite já especificados; não atesta implementação, aprovação de novos comportamentos ou integração real com Telegram.
