# Sabiá — Contrato operacional Telegram (2026-10-21)

**Referência oficial:** [Telegram Bot API 10.3](https://core.telegram.org/bots/api), publicada em 24/08/2026.

## 1. Garantia contra perda de updates

O Sabiá usa **transactional inbox**. Uma leitura `getUpdates` não é confirmação. Para cada update:

1. Abrir transação PostgreSQL e inserir `inbound_update` com `client_id`, `transport=telegram`, `update_id`, `raw_update jsonb NOT NULL`, `processing_status=accepted` e timestamps.
2. Garantir UNIQUE por `(client_id, transport, update_id)`. Update repetido reaproveita registro; nunca cria uma segunda execução.
3. Na mesma transação, avançar `transport_cursor` ao maior update da sequência **efetivamente persistida** e confirmar o commit.
4. Somente após commit, chamar `getUpdates(offset=last_update_id+1)`. Uma falha do commit impede avançar offset.
5. Worker usa claim transacional para normalizar `raw_update`, autorizar, criar a request e executar. Reinício recupera registros `accepted` e reconcilia `processing`, sem recriar operação concluída.

**Atomicidade remota:** não há transação distribuída com Telegram. Mesmo com inbox durável, resposta enviada mas não confirmada pode ser enviada novamente (semântica de entrega `at-least-once`). Side effects de scripts devem ser protegidos por chave idempotente (request_id) ou política explícita de execução não repetível.

**Crash tests:** antes de INSERT; após INSERT sem commit; após commit e antes de `getUpdates`; depois do claim; durante script; após envio e antes do commit do `message_id`. Em todos os casos, verificar recuperação sem perder update, sem duplicar a execução lógica e com política explícita para duplicação eventual de resposta.

**Retenção:** JSON bruto contém dados pessoais; acesso mínimo e limpeza somente depois de estado terminal, entregas/jogs reconciliados e retenção definida.

## 2. Botões e navegação

`/start` e `/menu` geram `InlineKeyboardMarkup` a partir das ações persistidas autorizadas ao usuário.

Exemplo:

```text
Sabiá
[ Automações ] [ Ferramentas ]
[ Status ]      [ Ajuda ]

Ferramentas
[ Converter vídeo ]
[ Voltar ] [ Início ]
```

Cada botão de categoria gera `callback_query`. Usar `answerCallbackQuery` para concluir o evento visual e `editMessageText` para trocar o conteúdo; se a mensagem não puder ser editada, enviar menu novo. `callback_data` usa identificador compacto de até 64 bytes; nunca contém caminho de script, parâmetros secretos ou permissão implícita. Verificar novamente token do cliente lógico, usuário, chat e permissão na ação antes de executar. Atualizar menus obsoletos em vez de aceitar callback antigo sem validação. `Voltar` retorna ao pai; `Início` à raiz; seleção de folha só dispara operação explicitamente autorizada.

## 3. Arquivos

Política de limite em **bytes decimais**:

| Etapa | Limite operacional |
|---|---:|
| Upload local integral por socket | 20.000.000 |
| Chunk de upload local | 5.000.000 |
| Upload local fracionado total | 50.000.000 |
| Envio comum de documento/vídeo pela Bot API pública | 50.000.000 |
| Download via `getFile` público | 20.000.000 |
| Foto por `sendPhoto` | Ver limite específico da API (inferior a 50 MB) |

`image/jpeg` e `image/png` compatíveis seguem `sendPhoto`; `video/mp4` compatível segue `sendVideo`; outros formatos seguem `sendDocument` quando aceitos. Conferir tamanho real antes do envio; a ausência de MIME não autoriza classificar por extensão; validar tipo/assinatura quando exigido por segurança. O fallback não deve contornar limites por método.

Se arquivo de entrada recebido no Telegram exceder o download público, devolver falha controlada e registrar no banco. Sem servidor Bot API local, o MVP **não promete baixar anexos recebidos de 20 a 50 MB**, mesmo que o armazenamento local suporte 50 MB.

## 4. Testes locais

Consultar [testes/telegram/README.md](testes/telegram/README.md). As suítes devem poder ser executadas separadamente com mocks HTTP da Bot API e PostgreSQL descartável. O repositório ainda não contém executável ou implementação em `/src`, portanto o material abaixo é especificação de testes a automatizar.
