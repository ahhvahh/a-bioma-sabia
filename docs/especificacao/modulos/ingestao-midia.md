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
- [CTR-0009 — Ingestão fracionada de mídia por Unix socket](../contratos/ingestao-midia-fracionada.md)
- [FLW-0008 — Limpeza de payloads de mídia transmitida](../fluxos/limpeza-midia.md)

## Responsabilidades

- abrir o Unix socket de ingestão simples;
- abrir um segundo Unix socket dedicado à ingestão fracionada;
- aceitar apenas conexões locais permitidas pelo sistema operacional;
- decodificar MessagePack;
- rejeitar payload simples acima de `20000000` bytes;
- rejeitar chunk acima de `5000000` bytes;
- validar versão e `request_id`;
- normalizar e validar `content_type` conforme CTR-0006, sem inferência por extensão ou inspeção dos bytes;
- criar `media_id` a partir dos metadados antes de receber chunks;
- persistir BLOB integral ou chunks ordenados e respectivos metadados;
- controlar `next_sequence_id` por mídia fracionada;
- rejeitar sequência pulada ou já recebida sem alterar os chunks persistidos;
- criar transmissão pendente na mesma unidade lógica;
- devolver ACK somente após commit;
- fornecer `media_id` à fila de entrega;
- impedir que binários sejam registrados em logs;
- recuperar transmissões pendentes sem filesystem temporário;
- executar limpeza de payloads entregues conforme FLW-0008 somente quando não houver transmissões `pending` ou `transmitting`.

## Entradas

Uploads integrais CTR-0008, chunks CTR-0009 e mídia recebida internamente pelo adaptador Telegram.

## Saídas

- `media_id` persistente;
- transmissão CTR-0007;
- ACK/erro CTR-0008;
- ACK/erro de chunk CTR-0009.

## Interfaces e contratos

- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)
- [CTR-0009 — Ingestão fracionada de mídia por Unix socket](../contratos/ingestao-midia-fracionada.md)

## Persistência

PostgreSQL armazena metadados, BLOB integral ou chunks ordenados. A transmissão referencia a mídia lógica por `media_id` quando ela estiver apta à entrega.

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
- payload integral acima de 20 MB é rejeitado com `media_too_large` e deve usar o canal fracionado;
- chunk acima de 5 MB é rejeitado com `chunk_too_large`;
- contrato permanece em `refinement` enquanto caminho/permissões dos sockets e demais dependências abertas não estiverem refinados.

## Implementação relacionada

Prevista para Media Ingest, armazenamento PostgreSQL e fila de mídia.
