# Ingestão fracionada de mídia por Unix socket

![CTR](https://img.shields.io/badge/CTR-CTR--0009-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir o segundo canal local de ingestão de mídia para arquivos maiores que o limite do CTR-0008, usando uma mídia lógica criada previamente no banco e chunks sequenciais persistidos individualmente.

## Dependências

- [ADR-0013 — Dois canais de ingestão de mídia e upload fracionado](../../adr/processamento/ingestao-midia-fracionada.md)
- [CTR-0006 — Mídia persistida](midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](transmissao-midia.md)
- [MOD-0006 — Segurança e autorização](../modulos/seguranca.md)

## Tipo

`mensagem`

## Transporte

O canal é um **Unix domain socket** distinto do socket CTR-0008.

Não existe listener TCP.

O caminho é configuração obrigatória e absoluta.

O mesmo socket aceita dois formatos de mensagem MessagePack:

1. abertura da mídia;
2. envio de chunk.

## Abertura da mídia

Antes de enviar chunks, o produtor envia os metadados do arquivo.

Objeto MessagePack:

- `version: integer` — versão do contrato; MVP usa `1`;
- `request_id: string` — identificador da requisição conhecido pelo produtor;
- `name: string` — nome lógico do arquivo;
- `content_type: string | null` — tipo declarado, quando conhecido.

O Sabiá:

1. valida a mensagem;
2. valida a existência de `request_id`;
3. cria o registro de mídia no banco;
4. confirma a transação;
5. devolve o identificador gerado pelo banco.

Resposta:

- `status: "accepted"`;
- `request_id: string`;
- `media_id: integer`;
- `next_sequence_id: integer`.

No MVP, a primeira sequência esperada é `1`.

A criação de `media_id` resolve a identidade do arquivo lógico. Dois arquivos com mesmo nome e mesma `request_id` recebem `media_id` diferentes.

## Envio de chunk

Depois de receber `media_id`, o produtor envia cada pedaço como:

- `media_id: integer`;
- `sequence_id: integer`;
- `data: binary`.

Não é necessário repetir nome, `request_id`, cliente ou destino em cada chunk.

## Tamanho dos chunks

Cada chunk pode conter no máximo:

`5000000` bytes

equivalentes a 5 MB decimais.

O último chunk pode ser menor.

Chunk maior que esse limite é recusado com `chunk_too_large` antes da persistência.

## Sequência

A sequência é estritamente crescente e inicia em `1`.

Para cada `media_id`, o Sabiá persiste qual é o próximo `sequence_id` esperado.

### Sequência correta

Quando `sequence_id == next_sequence_id`:

1. persistir o chunk;
2. confirmar a transação;
3. avançar `next_sequence_id`;
4. retornar ACK.

Resposta:

- `status: "accepted"`;
- `media_id: integer`;
- `sequence_id: integer`;
- `next_sequence_id: integer`.

### Sequência pulada

Quando `sequence_id > next_sequence_id`, o chunk não é persistido.

Resposta:

- `status: "error"`;
- `code: "sequence_gap"`;
- `media_id: integer`;
- `received_sequence_id: integer`;
- `expected_sequence_id: integer`;
- `message: string`.

O produtor pode corrigir imediatamente enviando a sequência esperada.

### Sequência já recebida

Quando `sequence_id < next_sequence_id`, o Sabiá considera que aquela posição já foi recebida e não grava outro chunk.

Resposta:

- `status: "error"`;
- `code: "sequence_already_received"`;
- `media_id: integer`;
- `received_sequence_id: integer`;
- `expected_sequence_id: integer`;
- `message: string`.

No MVP, não é feita comparação de hash ou conteúdo para decidir duplicidade. A duplicidade é determinada pela posição sequencial já persistida.

## Persistência

Cada chunk aceito é persistido em transação própria.

O ACK só é enviado depois do commit.

O serviço não precisa manter todos os chunks em memória simultaneamente.

A ordem do arquivo lógico é dada pela sequência persistida para o `media_id`.

## Outros erros

- `unsupported_version`;
- `unknown_request`;
- `unknown_media`;
- `invalid_message`;
- `chunk_too_large`;
- `sequence_gap`;
- `sequence_already_received`;
- `persistence_failed`;
- `not_authorized`.

## Entrega externa

Chunks não são enviados ao Telegram individualmente.

Quando a mídia lógica estiver completa, o Sabiá lê os chunks do `media_id` em ordem crescente e produz um stream contínuo para CTR-0007/Adaptador Telegram.

## Segurança

- socket acessível somente localmente;
- binários não aparecem em logs;
- conhecer `request_id` ou `media_id` não substitui autorização do Unix socket;
- `name` não representa caminho de filesystem;
- o produtor não escolhe cliente nem destino Telegram.

## BLOCKED

Antes de `refined`, ainda precisam ser definidos:

- como o produtor informa que não haverá mais chunks e que a mídia está completa;
- limite máximo do arquivo lógico completo;
- caminho, ownership, grupo e modo do socket.

## Critérios de aceite

- metadados criam um `media_id` persistente antes do primeiro chunk;
- arquivos com mesmo nome podem possuir `media_id` diferentes;
- cada chunk possui no máximo 5 MB;
- sequência inicia em 1 e é estritamente crescente;
- sequência pulada retorna `sequence_gap` com a sequência esperada;
- sequência já recebida retorna `sequence_already_received`;
- chunk válido é persistido antes do ACK;
- conteúdo pode ser reconstruído em ordem sem manter o arquivo inteiro em memória;
- arquivo lógico é entregue como um único arquivo ao transporte externo.
