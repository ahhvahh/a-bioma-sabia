# Ingestão fracionada de mídia

![FLW](https://img.shields.io/badge/FLW-FLW--0007-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Receber arquivos maiores em chunks persistidos sequencialmente e preparar sua posterior transmissão como um único arquivo lógico.

## Dependências

- [ADR-0013 — Dois canais de ingestão de mídia e upload fracionado](../../adr/processamento/ingestao-midia-fracionada.md)
- [MOD-0008 — Ingestão e armazenamento de mídia](../modulos/ingestao-midia.md)
- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](../contratos/transmissao-midia.md)
- [CTR-0009 — Ingestão fracionada de mídia por Unix socket](../contratos/ingestao-midia-fracionada.md)

## Gatilho

Produtor local conecta ao socket de ingestão fracionada e envia um chunk MessagePack.

## Pré-condições

- socket fracionado em execução;
- processo local autorizado;
- `request_id` conhecido;
- chunk com no máximo `5000000` bytes.

## Fluxo principal

1. aceitar conexão no socket fracionado;
2. decodificar um objeto MessagePack;
3. validar `version`, `request_id`, `name`, `sequence_id` e `data`;
4. rejeitar chunk acima de 5 MB;
5. consultar a requisição pelo `request_id`;
6. persistir o chunk e sua sequência;
7. confirmar a transação;
8. responder ACK do chunk;
9. encerrar a conexão;
10. repetir para os demais pedaços;
11. quando a mídia lógica estiver completa, disponibilizá-la para CTR-0007;
12. CTR-0007 lê os chunks em sequência e produz um único stream para o adaptador de destino.

## Fluxos alternativos

### Chunk fora do limite

Responder `chunk_too_large` antes da persistência.

### Requisição desconhecida

Responder `unknown_request`.

### Chunk recebido fora de ordem

O chunk pode ser persistido porque `sequence_id` define a ordem lógica. A política para lacunas e duplicidades permanece pendente.

### Restart

Chunks já confirmados permanecem no SQLite. A mídia incompleta continua persistida, mas não pode ser transmitida até que a regra de completude esteja satisfeita.

## Falhas e tratamento

Falha antes do commit nunca produz ACK de sucesso.

Falha de entrega ao Telegram não apaga os chunks necessários à recuperação da transmissão.

## Resultado

O arquivo lógico permanece representado por conteúdo persistido e ordenável sem necessidade de montar todo o arquivo em memória.

## BLOCKED

- identificação inequívoca do arquivo lógico quando houver mesmo `request_id` e `name`;
- indicação de último chunk/completude;
- regras para lacuna, repetição e continuidade de sequência;
- limite total do arquivo lógico.

## Critérios de aceite

- chunk de até 5 MB é persistido antes do ACK;
- chunks podem chegar fora de ordem e permanecem ordenáveis;
- restart não perde chunks confirmados;
- o Telegram recebe um único arquivo lógico;
- o processo não precisa alocar o arquivo completo em memória.

## Implementação relacionada

Media Chunk Ingest, SQLite, Media Store e Media Delivery Queue.
