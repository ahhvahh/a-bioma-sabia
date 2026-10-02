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

## Tipo

`interface`

## Entidade persistente

Cada conteúdo de mídia possui registro persistente no SQLite com, no mínimo:

- `media_id: integer` — identificador persistente;
- `request_id: string` — requisição à qual o conteúdo está associado;
- `name: string` — nome lógico do arquivo;
- `content_type: string | null` — tipo declarado/conhecido quando disponível;
- `size_bytes: integer` — tamanho persistido do BLOB;
- `data: blob` — conteúdo binário;
- `created_at: timestamp`.

`media_id` é a referência usada pelos demais contratos.

Nenhum contrato interno usa caminho de filesystem como identidade de mídia.

## Entrada por processador local

Scripts, aplicações e serviços enviam mídia por CTR-0008.

Depois de validar e persistir o BLOB, o Sabiá cria a correlação necessária para transmissão e devolve `media_id`.

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
- `size_bytes` corresponde ao conteúdo efetivamente persistido;
- conteúdo binário não aparece em logs;
- acesso ao SQLite é restrito à identidade operacional autorizada;
- produtor externo não escolhe `client_id`, transporte ou destino da mídia;
- `request_id` determina a correlação com a requisição original;
- nenhuma regra de deduplicação por `name`, hash ou conteúdo faz parte do MVP.

## BLOCKED

Ainda precisam ser definidos antes de `refined`:

- limite máximo de BLOB aceito;
- política de retenção do BLOB depois que todas as transmissões relacionadas forem concluídas;
- regra normativa para validar/determinar `content_type`.

## Compatibilidade

Novos transportes devem referenciar a mídia por `media_id`, sem depender de Telegram ou filesystem.

## Critérios de aceite

- mídia confirmada ao produtor permanece disponível após restart do serviço;
- vários arquivos podem ser associados à mesma requisição, inclusive com mesmo nome ou conteúdo;
- comando interno trabalha com referência e não com BLOB;
- transmissão recuperada após restart consegue ler o mesmo `media_id`;
- nenhum caminho temporário é necessário para mídia persistida.
