# Ingestão e armazenamento de mídia

![MOD](https://img.shields.io/badge/MOD-MOD--0008-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Receber mídia de produtores locais por Unix socket, validar a correlação, persistir metadados e BLOBs e criar transmissões duráveis para o cliente de origem.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0012 — Ingestão persistente de mídia por Unix socket](../../adr/processamento/ingestao-midia-socket-messagepack.md)
- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)

## Responsabilidades

- abrir o Unix socket de ingestão;
- aceitar apenas conexões locais permitidas pelo sistema operacional;
- decodificar MessagePack;
- validar versão e `request_id`;
- persistir BLOB e metadados;
- criar transmissão pendente na mesma unidade lógica;
- devolver ACK somente após commit;
- fornecer `media_id` à fila de entrega;
- impedir que binários sejam registrados em logs;
- recuperar transmissões pendentes sem filesystem temporário.

## Entradas

Uploads CTR-0008 e mídia recebida internamente pelo adaptador Telegram.

## Saídas

- `media_id` persistente;
- transmissão CTR-0007;
- ACK/erro CTR-0008.

## Interfaces e contratos

- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)

## Persistência

SQLite armazena metadados e BLOB da mídia. A transmissão referencia o conteúdo por `media_id`.

## Restrições

- não existe ingestão TCP;
- nenhum produtor escolhe destino Telegram;
- `request_id` precisa existir;
- conteúdo binário não aparece em logs;
- ACK só é emitido depois do commit.

## Critérios de aceite

- upload válido gera `media_id`;
- falha de persistência não produz ACK de sucesso;
- vários uploads podem compartilhar o mesmo `request_id`;
- restart mantém conteúdo confirmado;
- contrato permanece em `refinement` enquanto caminho/permissões do socket e limites máximos estiverem abertos.

## Implementação relacionada

Prevista para Media Ingest, armazenamento SQLite e fila de mídia.
