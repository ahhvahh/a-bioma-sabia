# Processamento assíncrono por processador registrado

![FLW](https://img.shields.io/badge/FLW-FLW--0005-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Executar um comando em processador assíncrono registrado, encaminhar progresso ao cliente e correlacionar mídia persistida produzida durante a execução.

## Dependências

- [MOD-0007 — Processadores assíncronos e transporte](../modulos/processadores-assincronos.md)
- [MOD-0008 — Ingestão e armazenamento de mídia](../modulos/ingestao-midia.md)
- [MOD-0004 — Jobs](../modulos/jobs.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)
- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)
- [CTR-0001 — Comando interno](../contratos/comando-interno.md)

## Gatilho

Command Router resolve uma operação cadastrada como processamento assíncrono.

## Pré-condições

- operação autorizada;
- processador cadastrado;
- `request_id`, `client_id` e `reply_context` disponíveis;
- capacidade de job disponível.

## Fluxo principal

1. o Sabiá cria/persiste o job e associa o `request_id`;
2. o Processor Registry resolve o processador cadastrado;
3. o Processor Transport envia `request_id`, comando e argumentos;
4. o processador inicia o trabalho;
5. a cada evento `loading`, o Sabiá valida o `request_id` e encaminha a mensagem ao cliente;
6. quando o processador produzir mídia, ele abre o socket CTR-0008 e envia um objeto MessagePack usando o mesmo `request_id`;
7. o Media Ingest persiste BLOB e metadados, cria a transmissão `pending` e retorna `media_id`;
8. a fila de mídia pode iniciar a entrega independentemente do canal de controle;
9. o processador pode repetir o upload para cada arquivo produzido;
10. o job permanece em execução;
11. o processador envia `finally` com a mensagem final;
12. o Sabiá persiste a resposta final;
13. o adaptador envia a mensagem final;
14. as transmissões de mídia seguem CTR-0007 até `delivered` ou `failed`.

## Fluxos alternativos

### loading repetido

Cada mensagem válida pode atualizar o cliente sem encerrar o job.

### vários arquivos

Cada arquivo é um upload independente CTR-0008. Todos podem usar o mesmo `request_id` e recebem `media_id` distinto.

### falha de upload de mídia

Se o upload não atingir commit, o produtor recebe erro e a mídia não é considerada aceita.

Se o commit ocorreu mas o ACK não chegou ao produtor, uma repetição pode criar duplicidade; a idempotência desse retry permanece pendente.

### falha de entrega ao Telegram

A mídia já persistida continua disponível. CTR-0007 mantém a transmissão `pending` ou `transmitting` e pode reenviar após restart.

### Processo termina sem finally

A requisição não é considerada concluída com sucesso apenas pelo término do processo. O tratamento depende da política de timeout de CTR-0005.

## Falhas e tratamento

- `request_id` desconhecido no controle ou na mídia: rejeitar;
- falha do Processor Transport: registrar falha de processamento;
- falha do Media Ingest: não confirmar o conteúdo;
- timeout sem `finally`: aplicar política de CTR-0005;
- falha de entrega externa não remove o BLOB persistido nem altera, por si só, o resultado do processamento.

## Resultado

O job possui estado final rastreável e toda mídia aceita possui `media_id` e transmissão persistente independente.

## Critérios de aceite

- `loading` chega ao cliente correto;
- `loading` não finaliza o job;
- `finally` é necessário para conclusão semântica normal;
- vários arquivos podem ser enviados pelo socket com o mesmo `request_id`;
- ACK de mídia ocorre somente após persistência;
- mídia pendente continua recuperável após restart do serviço;
- término do processo sem `finally` não é confundido com sucesso;
- canal de controle não transporta BLOB.

## Implementação relacionada

Processor Registry, Processor Transport, Media Ingest, Job Manager, SQLite e adaptadores de transporte.
