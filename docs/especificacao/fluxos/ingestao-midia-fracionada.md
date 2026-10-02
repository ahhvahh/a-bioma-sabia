# Ingestão fracionada de mídia

![FLW](https://img.shields.io/badge/FLW-FLW--0007-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Criar uma mídia lógica a partir dos metadados e receber seu conteúdo em chunks sequenciais persistidos individualmente.

## Dependências

- [ADR-0013 — Dois canais de ingestão de mídia e upload fracionado](../../adr/processamento/ingestao-midia-fracionada.md)
- [MOD-0008 — Ingestão e armazenamento de mídia](../modulos/ingestao-midia.md)
- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [CTR-0009 — Ingestão fracionada de mídia por Unix socket](../contratos/ingestao-midia-fracionada.md)

## Gatilho

Produtor local escolhe enviar um arquivo pelo socket fracionado. Para arquivos acima de 20 MB, esse canal é obrigatório; para arquivos menores ou iguais a 20 MB, é opcional.

## Pré-condições

- socket fracionado em execução;
- processo local autorizado;
- `request_id` conhecido;
- `total_bytes > 0`; não existe tamanho mínimo para usar o fluxo.

## Fluxo principal

1. produtor envia `version`, `request_id`, `name`, `content_type` e `total_bytes`;
2. Sabiá valida a requisição;
3. Sabiá valida `total_bytes` e cria a mídia com `received_bytes = 0` e `completed = false`;
4. commit gera `media_id`;
5. Sabiá responde `media_id` e `next_sequence_id = 1`;
6. produtor envia `media_id`, `sequence_id = 1` e `data`;
7. Sabiá valida tamanho e sequência;
8. calcula o novo total recebido e rejeita se ultrapassar `total_bytes`;
9. persiste o chunk, atualiza `received_bytes` e avança a próxima sequência esperada;
10. se `received_bytes == total_bytes`, marca `completed = true`;
11. responde o próximo `next_sequence_id`, `received_bytes`, `total_bytes` e `completed`;
12. produtor repete o envio enquanto `completed = false`;
13. quando `completed = true`, CTR-0007 pode iniciar sua transmissão;
14. CTR-0007 lê os chunks na sequência persistida e fornece um stream único ao adaptador.

## Fluxos alternativos

### Chunk acima de 5 MB

Responder `chunk_too_large`; não persistir.

### Mídia desconhecida

Responder `unknown_media`.

### Sequência pulada

Se o produtor enviar sequência maior que a esperada:

- não persistir;
- responder `sequence_gap`;
- informar `expected_sequence_id`.

### Sequência repetida

Se o produtor enviar sequência inferior à esperada:

- não persistir novamente;
- responder `sequence_already_received`;
- informar `expected_sequence_id`.

### Total excedido

Se um chunk faria o total persistido ultrapassar `total_bytes`:

- não persistir;
- responder `total_bytes_exceeded`;
- manter `received_bytes` e `next_sequence_id` inalterados.

### Restart

O banco preserva `media_id`, chunks confirmados e a próxima sequência esperada.

Após restart, o produtor pode continuar a partir da sequência indicada pelo estado persistido.

## Falhas e tratamento

Falha antes do commit não produz ACK.

A sequência nunca avança antes da persistência do chunk.

Falha de entrega ao Telegram não remove os chunks necessários ao recovery.

## Resultado

Um único `media_id` identifica todos os chunks do arquivo, independentemente do nome ou de outros arquivos associados à mesma requisição.

## BLOCKED

Ainda falta definir apenas o limite máximo permitido para `total_bytes`.

## Critérios de aceite

- abertura cria `media_id`;
- primeira sequência esperada é 1;
- ACK de chunk informa a próxima sequência;
- sequência pulada é recusada com valor esperado;
- sequência já recebida é recusada sem duplicar conteúdo;
- `completed` ocorre automaticamente quando `received_bytes == total_bytes`;
- não existe mensagem separada de finalização;
- restart preserva progresso;
- arquivo pode ser lido sequencialmente sem materialização integral em memória.

## Implementação relacionada

Media Chunk Ingest, SQLite, Media Store e Media Delivery Queue.
