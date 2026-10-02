# Adaptador Telegram

![MOD](https://img.shields.io/badge/MOD-MOD--0002-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Integrar cada cliente configurado com a Telegram Bot API sem acoplar o Core ao protocolo externo.

## Dependências

- [DSG-0001 — Contexto do Sabiá](../../desenho/contexto-sabia.md)
- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0005 — Telegram Long Polling](../../adr/telegram/long-polling.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../../adr/seguranca/menor-privilegio-e-autorizacao.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)

## Responsabilidades

- iniciar long polling por cliente habilitado;
- converter update em comando interno;
- receber imagens, vídeos e arquivos suportados e normalizá-los como anexos do comando;
- enviar imagens, vídeos e arquivos produzidos como resultado por streaming a partir da área temporária da requisição;
- aplicar o contexto correto do cliente;
- encaminhar identidade para autorização;
- converter respostas internas em mensagens Telegram;
- editar mensagem de progresso de job;
- enviar resposta/arquivo final pelo mesmo cliente e destino correlacionado à requisição;
- registrar correlações e entregas pendentes;
- durante shutdown, avisar todos os clientes ativos por seus destinos autorizados conhecidos.

## Entradas

Updates recebidos por `getUpdates`.

## Saídas

Comandos internos e chamadas de envio/edição de mensagens.

## Interfaces e contratos

- [CTR-0001 — Comando interno](../contratos/comando-interno.md)
- [CTR-0003 — Job](../contratos/job.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)
- [CTR-0006 — Mídia temporária por requisição](../contratos/midia-temporaria.md)

## Persistência

SQLite mantém a correlação entre requisição, cliente e destino de resposta. Mensagens prontas e não entregues voltam à etapa de envio após reinício.

Para cada cliente Telegram, o estado persistente deve registrar o maior `update_id` aceito para processamento. Um update somente pode ser considerado aceito após sua persistência transacional bem-sucedida no estado operacional.

Após persistir o update, a próxima chamada a `getUpdates` deve usar:

`offset = maior update_id persistido + 1`

A confirmação perante o Telegram, portanto, não depende da conclusão do comando. Depois da persistência, eventual falha ou reinício deve retomar o processamento a partir do estado local, sem solicitar novamente o mesmo update ao Telegram.

O processamento local deve tratar `update_id` como identificador idempotente por cliente, impedindo que uma repetição já persistida produza uma segunda execução da mesma requisição.

Respostas de saída devem permanecer persistidas como pendentes até que a Bot API retorne sucesso e o identificador da mensagem enviada seja persistido. Para mensagens, o `message_id` retornado no objeto `Message` é a evidência de entrega aceita pelo serviço remoto.

## Política de polling e retry de leitura

Cada cliente mantém seu próprio ciclo de long polling e seu próprio estado de backoff. Falhas de um cliente não suspendem nem atrasam os demais.

Para falhas temporárias de `getUpdates`, aplicar backoff exponencial:

`1s → 2s → 4s → 8s → 16s → 30s`

Regras:

- timeout de transporte, falha de rede e resposta `5xx`: avançar para a próxima etapa do backoff;
- `429` com `retry_after`: aguardar pelo menos o intervalo informado pelo Telegram, sem aplicar espera menor pelo backoff local;
- chamada `getUpdates` bem-sucedida, mesmo com lista vazia: resetar o backoff para `1s`;
- erro de autenticação ou autorização do cliente, incluindo `401` ou `403`: suspender o ciclo daquele cliente e registrar erro operacional; não repetir indefinidamente;
- falha de polling nunca altera o `offset`;
- o uso de pequeno jitter para reduzir reconexões simultâneas é permitido como detalhe de implementação, sem alterar os limites normativos acima.

## Política de envio e retry

A entrega pelo Telegram usa semântica `at-least-once`.

- sucesso da Bot API: persistir `message_id` e somente então marcar a entrega como `delivered`;
- erro com `retry_after`: manter `pending` e aguardar pelo menos o intervalo indicado antes da próxima tentativa;
- timeout, desconexão ou falha temporária sem confirmação: manter `pending` e permitir nova tentativa;
- erro permanente de requisição: marcar a entrega como `failed`, preservando código e descrição para observabilidade;
- após reinício, entregas textuais `pending` retornam à etapa de envio;
- entregas de mídia baseadas em `/tmp/sabia/media/<request_id>/` não sobrevivem a restart e não são reenviadas após a limpeza de startup;
- em falha ambígua após o envio, priorizar eventual entrega: nova tentativa é permitida mesmo que isso possa produzir duplicidade rara.

A aplicação não considera a ausência de resposta da Bot API como prova de que a mensagem não foi entregue.

## Restrições

- long polling e webhook não são usados simultaneamente para o mesmo cliente;
- tokens não aparecem em logs;
- nenhum teste depende da API real;
- uma resposta de um cliente não pode ser entregue usando identidade de outro cliente;
- o `offset` nunca avança antes da persistência bem-sucedida do update correspondente;
- uma entrega não pode ser marcada como `delivered` antes da persistência da confirmação remota;
- arquivo temporário só pode ser removido após transmissão completa e confirmação remota;
- após confirmação de mídia, o arquivo deve ser removido imediatamente;
- conteúdo binário de entrada ou saída não pode ser registrado em logs;
- referências de mídia seguem CTR-0006;
- o adaptador nunca aceita para envio arquivo fora de `/tmp/sabia/media/<request_id>/`;
- lifecycle, tipo de mídia e limites permanecem em `refinement`.

## Critérios de aceite

- clientes habilitados funcionam independentemente;
- resposta é enviada pelo cliente que recebeu a requisição;
- todos os clientes ativos recebem tentativa de aviso no shutdown;
- mensagem textual pendente pode voltar à etapa de envio após reinício;
- mídia temporária pendente não é recuperável após restart;
- update persistido não é executado novamente após reinício ou repetição da Bot API;
- falha antes da persistência não avança o `offset`;
- `retry_after` é respeitado quando fornecido;
- timeout ou falha ambígua não remove a resposta da fila de entrega;
- sucesso de envio persiste `message_id` antes de concluir a entrega;
- falha temporária de `getUpdates` usa backoff exponencial até o máximo de `30s`;
- sucesso de `getUpdates` reseta o backoff para `1s`;
- `429` respeita `retry_after`;
- falha de autenticação/autorização suspende somente o cliente afetado;
- falha de polling não altera o `offset`;
- imagens e vídeos recebidos podem ser associados ao `request_id` sem expor objetos da Bot API ao Core;
- imagens e vídeos de resultado são referenciados por `{name, path}` e enviados por streaming;
- múltiplos arquivos da mesma requisição são enviados por eventos `content` independentes.

## Referência externa

- Telegram Bot API: https://core.telegram.org/bots/api

## Implementação relacionada

Prevista para o pacote interno de integração Telegram.
