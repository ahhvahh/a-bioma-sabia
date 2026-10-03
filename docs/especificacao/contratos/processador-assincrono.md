# Protocolo de processador assíncrono

![CTR](https://img.shields.io/badge/CTR-CTR--0005-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Definir o protocolo de controle entre o Sabiá e scripts Bash registrados que atuam como processadores assíncronos no primeiro MVP.

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

No primeiro MVP, o protocolo é específico para scripts Bash:

- o `stdin` recebe somente o `request_id` UUID v4 como uma linha UTF-8 terminada em `\n`;
- argumentos da operação são fornecidos ao processo via argumentos de linha de comando conforme CTR-0002;
- o `stdout` é reservado exclusivamente aos eventos deste contrato em JSON Lines;
- o `stderr` é reservado para diagnóstico/log operacional e não participa do protocolo;
- aplicações dedicadas e serviços por socket ficam fora do MVP.

O payload de controle não usa MessagePack nem Protobuf.

Exemplo de entrada no `stdin`:

```text
550e8400-e29b-41d4-a716-446655440000
```

Exemplos de saída no `stdout`:

```json
{"request_id":"req-123","status":"loading","message":"50%"}
{"request_id":"req-123","status":"finally","message":"Concluído"}
```

Cada linha emitida em `stdout` deve conter exatamente um objeto JSON completo. Texto não JSON em `stdout` viola o protocolo.

## Entrada do Sabiá para o processador

Toda execução assíncrona recebe:

- no `stdin`: `request_id` UUID v4 canônico gerado pelo Sabiá, em uma única linha;
- em `argv`: os argumentos já validados pela operação, conforme CTR-0002.

O script não gera nem substitui o `request_id`.

Quando uma operação precisar consumir mídia de entrada, a referência persistente continua sendo `media_id` conforme CTR-0006/CTR-0001; a forma concreta de passar argumentos da operação continua pertencendo ao contrato da própria operação.

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

Qualquer script Bash registrado que possua o `request_id` e autorização local para o socket de mídia pode publicar um ou vários arquivos por CTR-0008.

Mídia e eventos de controle são canais distintos:

- CTR-0005: progresso e finalização;
- CTR-0008: conteúdo binário.

O Sabiá correlaciona ambos pelo mesmo `request_id`.

## Relação com jobs

`loading` representa progresso sem criar estado terminal.

`finally` encerra semanticamente o processamento normal.

Uploads de mídia podem ocorrer antes de `finally` e geram transmissões independentes. A entrega da mídia não altera por si só o estado terminal do job.

## Timeout de execução

Toda execução CTR-0005 possui timeout absoluto.

No MVP, o valor inicial é **2 horas** por execução.

Regras:

- o contador começa quando o Sabiá entrega a requisição ao processador;
- eventos `loading` não reiniciam nem prorrogam o prazo;
- receber `finally` válido antes do prazo encerra normalmente o processamento;
- atingir o prazo sem `finally` encerra a execução com `processor_timeout`;
- o job associado realiza `running → timeout`;
- após o job atingir estado terminal, eventos tardios daquele processamento não alteram o resultado;
- o Sabiá solicita o encerramento/cancelamento do mecanismo de transporte associado ao atingir o timeout;
- se processo, conexão ou transporte encerrar antes do prazo sem `finally`, o processamento termina em falha de transporte e não aguarda as 2 horas restantes.

O timeout é de duração total da execução, não de inatividade.

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

O MVP suporta somente scripts Bash. A inclusão futura de aplicações dedicadas ou serviços por socket exigirá extensão explícita deste contrato, sem alterar implicitamente o protocolo dos scripts existentes.

Mídia sempre usa a fronteira específica de CTR-0008.

## Critérios de aceite

- toda execução recebe `request_id` gerado pelo Sabiá;
- todo evento devolve o mesmo `request_id`;
- `loading` pode chegar ao cliente sem encerrar o job;
- `finally` encerra semanticamente o processamento;
- mídia não trafega no protocolo de controle;
- vários arquivos podem ser associados ao mesmo `request_id` via CTR-0008;
- o script recebe somente `request_id` pelo stdin, em uma linha UTF-8;
- argumentos da operação chegam via argv;
- stdout contém somente JSON Lines UTF-8, um objeto por linha;
- serviços por socket e aplicações dedicadas não fazem parte do MVP;
- toda execução respeita timeout absoluto de 2 horas no MVP;
- `loading` não renova o timeout;
- ausência de `finally` até o limite resulta em `processor_timeout` e job `timeout`.
