# Protocolo de processador assíncrono

![CTR](https://img.shields.io/badge/CTR-CTR--0005-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir o protocolo de controle entre o Sabiá e processadores assíncronos registrados, independentemente de o processador ser script Bash, aplicação/executável ou serviço acessível por socket.

Conteúdo binário não trafega neste contrato; mídia usa CTR-0008.

## Dependências

- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](../../adr/processamento/processadores-assincronos-registrados.md)
- [ADR-0012 — Ingestão persistente de mídia por Unix socket](../../adr/processamento/ingestao-midia-socket-messagepack.md)
- [CTR-0001 — Comando interno](comando-interno.md)
- [CTR-0003 — Job](job.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](ingestao-midia-messagepack.md)
- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)

## Tipo

`mensagem`

## Formato e framing

O canal de controle usa **JSON Lines (JSONL/NDJSON)** em UTF-8.

Cada mensagem lógica é serializada como um único objeto JSON em uma única linha, terminada por `\n`. O delimitador de mensagem é a quebra de linha; não há prefixo binário de tamanho.

O mesmo framing é usado para:

- stdin/stdout de scripts e aplicações executáveis;
- streams de serviços acessíveis por socket.

O payload de controle não usa MessagePack nem Protobuf.

Exemplo de entrada:

```json
{"request_id":"req-123","command":"convert","arguments":["a","b"]}
```

Exemplos de saída:

```json
{"request_id":"req-123","status":"loading","message":"50%"}
{"request_id":"req-123","status":"finally","message":"Concluído"}
```

Para scripts/aplicações:

- o Sabiá envia a requisição pelo `stdin` como uma linha JSON;
- o processador publica eventos CTR-0005 no `stdout`, uma linha JSON por evento;
- `stderr` fica reservado para diagnóstico/log operacional e não participa do protocolo.

Para serviços/socket:

- requisição e eventos usam o mesmo envelope JSON e o mesmo delimitador `\n`;
- uma conexão pode transportar uma sequência de mensagens, sempre uma por linha;
- objeto JSON inválido ou linha que não corresponde ao contrato é erro de transporte/protocolo.

## Entrada do Sabiá para o processador

Toda execução assíncrona recebe, no mínimo:

- `request_id: string`;
- `command: string`;
- `arguments: string[]`.

Quando existirem anexos de entrada, o processador recebe referências por `media_id` conforme CTR-0006/CTR-0001.

O processador não gera nem substitui o `request_id`.

## Eventos do processador para o Sabiá

Cada evento de controle contém:

- `request_id: string`;
- `status: loading | finally`;
- `message: string`.

### loading

Representa atualização intermediária.

- pode ocorrer zero ou mais vezes;
- não encerra o job;
- a mensagem pode ser encaminhada ao cliente correlacionado.

### finally

Representa a finalização semântica da requisição.

- ocorre no máximo uma vez como término normal;
- contém a mensagem final;
- não carrega mídia nem binário.

Depois de um `finally` válido, novos eventos de controle não alteram o resultado da requisição.

## Mídia produzida

Qualquer script, aplicação ou serviço que possua o `request_id` e autorização local para o socket de mídia pode publicar um ou vários arquivos por CTR-0008.

Mídia e eventos de controle são canais distintos:

- CTR-0005: progresso e finalização;
- CTR-0008: conteúdo binário.

O Sabiá correlaciona ambos pelo mesmo `request_id`.

## Relação com jobs

`loading` representa progresso sem criar estado terminal.

`finally` encerra semanticamente o processamento normal.

Uploads de mídia podem ocorrer antes de `finally` e geram transmissões independentes. A entrega da mídia não altera por si só o estado terminal do job.

## Erros

- `unknown_request`;
- `invalid_status`;
- `processor_timeout`;
- `transport_failure`.

Erros de mídia pertencem a CTR-0008/CTR-0007.

## Regras e restrições

- somente processadores cadastrados recebem requisições;
- eventos não acessam diretamente Telegram;
- toda correlação usa `request_id`;
- `loading` não finaliza processamento;
- `finally` é terminal;
- conteúdo binário não trafega neste protocolo;
- stdout/stderr de CTR-0002 não substituem este protocolo assíncrono.

## Compatibilidade

Scripts, aplicações e serviços usam o mesmo formato lógico e framing JSON Lines. O Processor Transport adapta apenas o mecanismo concreto de transporte, preservando o mesmo contrato.

Mídia sempre usa a fronteira específica de CTR-0008.

## BLOCKED

Ainda precisam ser definidos antes de `refined`:

- política de timeout sem `finally`.

## Critérios de aceite

- toda execução recebe `request_id` gerado pelo Sabiá;
- todo evento devolve o mesmo `request_id`;
- `loading` pode chegar ao cliente sem encerrar o job;
- `finally` encerra semanticamente o processamento;
- mídia não trafega no protocolo de controle;
- vários arquivos podem ser associados ao mesmo `request_id` via CTR-0008;
- scripts/aplicações e serviços/socket usam JSON Lines UTF-8, com um objeto por linha.
