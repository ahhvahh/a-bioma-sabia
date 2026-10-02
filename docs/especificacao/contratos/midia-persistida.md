# Mídia persistida

![CTR](https://img.shields.io/badge/CTR-CTR--0006-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir a representação persistente de imagens, vídeos e outros arquivos usados pelo Sabiá, eliminando dependência de caminhos temporários de filesystem.

## Dependências

- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0012 — Ingestão persistente de mídia por Unix socket](../../adr/processamento/ingestao-midia-socket-messagepack.md)
- [CTR-0001 — Comando interno](comando-interno.md)
- [CTR-0007 — Transmissão persistente de mídia](transmissao-midia.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](ingestao-midia-messagepack.md)
- [CTR-0009 — Ingestão fracionada de mídia por Unix socket](ingestao-midia-fracionada.md)
- [FLW-0008 — Limpeza de payloads de mídia transmitida](../fluxos/limpeza-midia.md)

## Tipo

`interface`

## Entidade persistente

Cada conteúdo de mídia possui registro persistente no SQLite com, no mínimo:

- `media_id: integer` — identificador persistente;
- `request_id: string` — requisição à qual o conteúdo está associado;
- `name: string` — nome lógico do arquivo;
- `content_type: string | null` — tipo declarado/conhecido quando disponível;
- `size_bytes: integer` — tamanho efetivamente persistido até o momento;
- `total_bytes: integer` — tamanho total esperado do conteúdo;
- `received_bytes: integer` — quantidade total de bytes já persistida;
- `data: blob | null` — conteúdo binário quando a mídia foi recebida integralmente pelo canal simples;
- `created_at: timestamp`;
- `storage_mode: inline | chunked`;
- `next_sequence_id: integer | null` — próxima sequência esperada quando `storage_mode = chunked`;
- `completed: boolean` — indica se `received_bytes == total_bytes` e a mídia está apta à transmissão.

`media_id` é a referência usada pelos demais contratos.

Nenhum contrato interno usa caminho de filesystem como identidade de mídia.

## Entrada por processador local

Scripts, aplicações e serviços enviam mídia por CTR-0008.

Depois de validar e persistir o BLOB integral pelo CTR-0008, o Sabiá cria a correlação necessária para transmissão e devolve `media_id`.

Quando o produtor escolher armazenamento fracionado, usa CTR-0009 independentemente do tamanho total. Para conteúdo acima de 20 MB, CTR-0009 é obrigatório porque CTR-0008 recusa o upload integral. A abertura cria primeiro o registro de mídia com `total_bytes`, retorna `media_id`, inicializa `received_bytes = 0` e `next_sequence_id = 1`. Cada chunk é persistido separadamente e ordenado por `sequence_id`. Quando `received_bytes == total_bytes`, `completed` passa automaticamente para `true`.

Uma mesma `request_id` pode possuir vários registros de mídia. Nome e conteúdo não possuem restrição de unicidade: arquivos iguais recebidos mais de uma vez são persistidos como registros independentes.

## Entrada pelo Telegram

Mídia recebida do Telegram também deve ser normalizada para este modelo persistente antes de ser disponibilizada a operações que dependam de conteúdo binário.

O Core recebe referências por `media_id`, não objetos da Bot API.

## Referência no comando interno

Anexos usam:

- `media_id: integer`;
- `name: string`;
- `content_type: string | null`.

O binário permanece no armazenamento persistente e não é copiado para o envelope de comando.

## Saída

Resultados binários usam `media_id` para referenciar conteúdo já persistido.

A entrega ao Telegram é controlada por CTR-0007.

## Erros

- `media_not_found`;
- `media_persistence_failed`;
- `media_too_large`;
- `media_corrupt`.

## Regras e restrições

- BLOB só é considerado disponível após commit;
- cada BLOB integral aceito pelo canal simples possui no máximo `20000000` bytes;
- chunks aceitos pelo canal fracionado possuem no máximo `5000000` bytes;
- conteúdo fracionado é lido em ordem de `sequence_id` sem exigir montagem integral em memória;
- `next_sequence_id` só avança após commit do chunk esperado;
- chunks com sequência pulada ou já recebida não alteram o conteúdo;
- `size_bytes` e `received_bytes` correspondem ao conteúdo efetivamente persistido;
- `received_bytes` nunca pode ultrapassar `total_bytes`;
- mídia fracionada só pode ser transmitida quando `received_bytes == total_bytes`;
- conteúdo binário não aparece em logs;
- acesso ao SQLite é restrito à identidade operacional autorizada;
- produtor externo não escolhe `client_id`, transporte ou destino da mídia;
- `request_id` determina a correlação com a requisição original;
- nenhuma regra de deduplicação por `name`, hash ou conteúdo faz parte do MVP.

## Retenção do payload

Conteúdo binário de mídia já entregue pode ser removido por um processo de limpeza.

A limpeza pode remover:

- `data` de mídia inline já entregue;
- registros de chunks pertencentes a mídia já entregue.

A limpeza não remove metadados mínimos necessários à rastreabilidade, incluindo `media_id`, `request_id`, nome, tipo, tamanho e estado de entrega.

A limpeza nunca pode ocorrer enquanto existir qualquer transmissão em `pending` ou `transmitting`.

## BLOCKED

Ainda precisam ser definidos antes de `refined`:

- regra normativa para validar/determinar `content_type`;
- limite máximo de `total_bytes` para mídia fracionada, conforme CTR-0009.

## Compatibilidade

Novos transportes devem referenciar a mídia por `media_id`, sem depender de Telegram ou filesystem.

## Critérios de aceite

- mídia confirmada ao produtor permanece disponível após restart do serviço;
- vários arquivos podem ser associados à mesma requisição, inclusive com mesmo nome ou conteúdo;
- comando interno trabalha com referência e não com BLOB;
- transmissão recuperada após restart consegue ler o mesmo `media_id`;
- nenhum caminho temporário é necessário para mídia persistida;
- mídia integral acima de 20 MB é recusada pelo canal simples e deve usar o canal fracionado;
- mídia de até 20 MB também pode usar `storage_mode = chunked`;
- chunk acima de 5 MB é recusado pelo canal fracionado;
- mídia fracionada fica `completed` automaticamente ao atingir `total_bytes`;
- payload entregue pode ser limpo somente sem transmissões `pending` ou `transmitting`.
