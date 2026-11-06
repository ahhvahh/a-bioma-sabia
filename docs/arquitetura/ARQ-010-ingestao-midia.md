<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ARQ-010 — Ingestão e armazenamento de mídia

**ID:** `ARQ-010` · **Origem histórica:** `MOD-0008`  
**Estado original:** `refinement` · **Implementação:** não verificada nesta branch  
**Classificação:** revisão documental; itens em `refinement` não foram aprovados por esta migração.

## Objetivo

Receber mídia de produtores locais por Unix socket, validar a correlação, persistir metadados e BLOBs e criar transmissões duráveis para o cliente de origem.

## Dependências

- [ARQ-002 (DSG-0002) — Componentes do Sabiá Core](ARQ-002-componentes-core.md)
- [DEC-009 (ADR-0009) — Persistência do estado operacional](../decisoes/DEC-009-postgresql-estado-operacional.md)
- [DEC-012 (ADR-0012) — Ingestão persistente de mídia por Unix socket](../decisoes/DEC-012-ingestao-midia-socket-messagepack.md)
- [CONTR-006 (CTR-0006) — Mídia persistida](../contratos/CONTR-006-midia-persistida.md)
- [CONTR-007 (CTR-0007) — Transmissão persistente de mídia](../contratos/CONTR-007-transmissao-midia.md)
- [CONTR-008 (CTR-0008) — Ingestão de mídia por Unix socket e MessagePack](../contratos/CONTR-008-ingestao-midia-messagepack.md)
- [CONTR-009 (CTR-0009) — Ingestão fracionada de mídia por Unix socket](../contratos/CONTR-009-ingestao-midia-fracionada.md)
- [FLUX-008 (FLW-0008) — Limpeza de payloads de mídia transmitida](../fluxos/FLUX-008-limpeza-midia.md)

## Responsabilidades

- abrir o Unix socket de ingestão simples;
- abrir um segundo Unix socket dedicado à ingestão fracionada;
- aceitar apenas conexões locais do usuário `sabia` ou de processos pertencentes ao grupo Linux `abioma`, conforme permissões `0660` dos sockets;
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

- [CONTR-006 (CTR-0006) — Mídia persistida](../contratos/CONTR-006-midia-persistida.md)
- [CONTR-007 (CTR-0007) — Transmissão persistente de mídia](../contratos/CONTR-007-transmissao-midia.md)
- [CONTR-008 (CTR-0008) — Ingestão de mídia por Unix socket e MessagePack](../contratos/CONTR-008-ingestao-midia-messagepack.md)
- [CONTR-009 (CTR-0009) — Ingestão fracionada de mídia por Unix socket](../contratos/CONTR-009-ingestao-midia-fracionada.md)

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
- contrato permanece em `refinement` enquanto os caminhos normativos dos sockets e demais dependências abertas não estiverem refinados.

## Implementação relacionada

Prevista para Media Ingest, armazenamento PostgreSQL e fila de mídia.

## Pendências e integração

O grupo `sabia:abioma` no texto histórico conflita com a proposta posterior `absabia` para UDS da aplicação; os caminhos normativos dos sockets não foram fechados. O limite total fracionado é `50000000` bytes conforme [CONTR-009](../contratos/CONTR-009-ingestao-midia-fracionada.md).

## Rastreabilidade

Documento original preservado: [MOD-0008 — Ingestão e armazenamento de mídia](../especificacao/modulos/ingestao-midia.md).
