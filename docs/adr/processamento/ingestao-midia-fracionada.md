# Dois canais de ingestão de mídia e upload fracionado

![ADR](https://img.shields.io/badge/ADR-ADR--0013-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

A ingestão simples por Unix socket e MessagePack é suficiente para arquivos pequenos, mas manter arquivos maiores em um único campo binário aumenta uso de memória e dificulta persistência progressiva.

O Sabiá também precisa continuar isolando responsabilidades para que o canal simples permaneça fácil de testar.

## Problema

Definir como receber arquivos maiores sem ampliar indefinidamente o payload do socket simples e sem exigir que o arquivo inteiro seja mantido em memória antes de persistir.

## Restrições

- arquivos de até 20 MB devem continuar usando o canal simples;
- arquivos maiores devem possuir canal Unix socket próprio;
- o canal fracionado usa MessagePack;
- cada pedaço deve ser persistido assim que recebido;
- pedaços pertencentes ao mesmo arquivo precisam ser ordenáveis;
- a correlação com cliente e destino continua usando o `request_id` já existente;
- o Telegram não oferece no Bot API um protocolo de upload por chunks equivalente ao protocolo interno do Sabiá; a entrega externa usa o arquivo lógico completo.

## Opções consideradas

### Aumentar o limite do socket simples

Mantém um único contrato, mas exige payloads maiores e concentra ingestão simples e grande no mesmo canal.

### Segundo socket com chunks persistidos

Mantém o caminho simples pequeno e permite persistência progressiva de arquivos maiores.

## Decisão

O Sabiá terá dois Unix domain sockets de mídia:

1. **Media Ingest** — arquivo integral em uma mensagem MessagePack, limitado a `20000000` bytes;
2. **Media Chunk Ingest** — arquivo enviado em partes independentes e ordenáveis.

O tamanho normativo máximo de cada chunk é `5000000` bytes (5 MB decimais). O último chunk pode ser menor.

Antes dos chunks, o produtor envia os metadados do arquivo usando `request_id`, `name`, tipo conhecido e `total_bytes`. O Sabiá cria o objeto de mídia e devolve o `media_id` gerado pelo banco.

Depois disso, cada chunk contém somente:

- `media_id`;
- `sequence_id`;
- `data`.

O Telegram `message_id` não é exposto ao produtor.

A sequência inicia em `1` e deve ser estritamente crescente. Cada chunk é persistido assim que é aceito. O Sabiá devolve a próxima sequência esperada em cada ACK. A mídia é considerada completa automaticamente quando a soma dos bytes persistidos atinge exatamente `total_bytes`.

A transmissão ao Telegram não envia cada chunk como uma mensagem independente. Quando o arquivo lógico estiver completo, o Sabiá lê os chunks persistidos em ordem e fornece um stream contínuo ao upload multipart do adaptador Telegram, sem precisar materializar o arquivo inteiro em memória.

## Justificativa

A separação mantém o socket simples previsível, torna arquivos maiores persistíveis incrementalmente e mantém a responsabilidade de montagem/streaming dentro do Sabiá.

## Consequências

- CTR-0008 passa a aceitar no máximo 20 MB;
- será necessário um novo contrato para o socket fracionado;
- a persistência de mídia passa a suportar registros de chunks ordenados;
- a fila de transmissão precisa conseguir ler conteúdo por chunks;
- a identidade do arquivo lógico é resolvida pelo `media_id` gerado na abertura;
- lacunas e sequências repetidas são recusadas com a próxima sequência esperada;
- não existe mensagem separada de término; `received_bytes == total_bytes` define completude;
- o limite máximo permitido para `total_bytes` precisa respeitar o transporte de destino.

## Dependências

- [ADR-0012 — Ingestão persistente de mídia por Unix socket](ingestao-midia-socket-messagepack.md)
- [CTR-0006 — Mídia persistida](../../especificacao/contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../../especificacao/contratos/transmissao-midia.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../../especificacao/contratos/ingestao-midia-messagepack.md)

## Critérios de validação

- arquivos de até 20 MB usam o socket simples;
- payload simples acima de 20 MB é recusado;
- arquivo grande pode ser persistido em chunks de até 5 MB;
- vários chunks são associados ao mesmo `media_id`;
- dois arquivos com mesmo nome podem coexistir porque recebem `media_id` distintos;
- sequência persistida permite reconstrução ordenada;
- sequência pulada ou repetida é detectada imediatamente;
- completude é determinada por `total_bytes` sem mensagem adicional;
- Telegram recebe um único arquivo lógico e não os chunks individualmente;
- a montagem não exige manter o arquivo completo em memória.
