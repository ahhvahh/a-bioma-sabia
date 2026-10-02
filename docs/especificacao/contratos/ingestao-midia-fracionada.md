# Ingestão fracionada de mídia por Unix socket

![CTR](https://img.shields.io/badge/CTR-CTR--0009-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir o segundo canal local de ingestão de mídia para arquivos que não devem ser enviados integralmente pelo CTR-0008.

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

## Codificação

Cada conexão envia um objeto MessagePack correspondente a um único chunk e recebe um objeto MessagePack de resposta.

## Entrada

Objeto MessagePack:

- `version: integer` — versão do contrato; MVP usa `1`;
- `request_id: string` — identificador da requisição conhecido pelo produtor;
- `name: string` — nome lógico do arquivo;
- `sequence_id: integer` — identificador sequencial do pedaço;
- `data: binary` — conteúdo do pedaço.

O contrato usa `request_id` como o identificador da mensagem/requisição. O Telegram `message_id` não atravessa esta interface.

## Tamanho dos chunks

Cada chunk pode conter no máximo:

`5000000` bytes

equivalentes a 5 MB decimais.

O último chunk pode ser menor.

Chunk maior que esse limite deve ser recusado com `chunk_too_large` antes da persistência.

## Persistência

Para cada chunk aceito, o Sabiá deve:

1. validar versão e estrutura MessagePack;
2. validar a existência de `request_id`;
3. validar o limite de 5 MB;
4. persistir imediatamente `request_id`, `name`, `sequence_id` e `data`;
5. confirmar a transação;
6. somente depois retornar ACK.

A ordem lógica do conteúdo é definida por `sequence_id`.

O serviço não precisa manter todos os chunks em memória ao mesmo tempo.

## Resposta de sucesso

Objeto MessagePack:

- `status: "accepted"`;
- `request_id: string`;
- `name: string`;
- `sequence_id: integer`.

## Resposta de erro

Objeto MessagePack:

- `status: "error"`;
- `code: string`;
- `message: string`.

Códigos mínimos:

- `unsupported_version`;
- `unknown_request`;
- `invalid_message`;
- `chunk_too_large`;
- `persistence_failed`;
- `not_authorized`.

## Entrega externa

Chunks não são enviados ao Telegram como mensagens ou arquivos independentes.

Quando o arquivo lógico estiver completo, o Sabiá deve ler os chunks persistidos em ordem crescente de `sequence_id` e produzir um stream contínuo para CTR-0007/Adaptador Telegram.

A persistência por chunks existe para evitar materialização integral do arquivo em memória.

## Segurança

- socket acessível somente localmente;
- binários não aparecem em logs;
- conhecer `request_id` não substitui a autorização do Unix socket;
- `name` não representa caminho de filesystem;
- o produtor não escolhe cliente nem destino Telegram.

## BLOCKED

Antes de `refined`, ainda precisam ser definidos:

- como identificar de forma inequívoca dois arquivos fracionados com o mesmo `request_id` e `name`;
- como indicar que o arquivo lógico está completo;
- domínio/valor inicial e regras de continuidade de `sequence_id`;
- comportamento para chunk ausente, duplicado ou fora de ordem;
- limite máximo do arquivo lógico completo;
- caminho, ownership, grupo e modo do socket.

## Critérios de aceite

- o socket fracionado é diferente do socket simples;
- cada chunk possui no máximo 5 MB;
- cada chunk é persistido antes do ACK;
- chunks podem chegar sem manter o arquivo inteiro em memória;
- conteúdo é reconstruído em ordem de sequência;
- arquivo lógico é entregue como um único arquivo ao transporte externo;
- lacunas de identidade/completude acima permanecem `BLOCKED`.
