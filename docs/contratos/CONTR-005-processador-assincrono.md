<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# CONTR-005 — Protocolo de processador assíncrono

**ID documental:** `CONTR-005` · **ID histórico:** `CTR-0005`  
**Estado da especificação de origem:** `refined` · **Implementação:** não verificada  
**Tipo:** contrato técnico do Sabiá; a classificação `REVISADO` indica revisão documental, não aprovação funcional nem código executado.

## Objetivo

Definir o protocolo de controle entre o Sabiá e scripts Bash registrados que atuam como processadores assíncronos no primeiro MVP.

Conteúdo binário não trafega neste contrato; mídia usa CTR-0008.

## Dependências

- [DEC-011 (ADR-0011) — Processadores assíncronos registrados e transporte de progresso](../decisoes/DEC-011-processadores-assincronos-registrados.md)
- [DEC-012 (ADR-0012) — Ingestão persistente de mídia por Unix socket](../decisoes/DEC-012-ingestao-midia-socket-messagepack.md)
- [CONTR-001 (CTR-0001) — Comando interno](CONTR-001-comando-interno.md)
- [CONTR-003 (CTR-0003) — Job](CONTR-003-job.md)
- [CONTR-008 (CTR-0008) — Ingestão de mídia por Unix socket e MessagePack](CONTR-008-ingestao-midia-messagepack.md)
- [ARQ-002 (DSG-0002) — Componentes do Sabiá Core](../arquitetura/ARQ-002-componentes-core.md)

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
{"request_id":"550e8400-e29b-41d4-a716-446655440000","status":"loading","message":"50%"}
{"request_id":"550e8400-e29b-41d4-a716-446655440000","status":"finally","message":"Concluído"}
```

Cada linha emitida em `stdout` deve conter exatamente um objeto JSON completo. Texto não JSON em `stdout` viola o protocolo. O Sabiá consome cada linha assim que ela é emitida, para que eventos `loading` possam ser processados durante a execução sem aguardar o término do script.

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
- quando o script atua como processador assíncrono, o stdout é consumido incrementalmente como protocolo CTR-0005; não é tratado como resultado textual livre de CTR-0002;
- stderr permanece o canal separado de diagnóstico definido por CTR-0002.

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

## Limitações da revisão

**Pendência de implementação:** o controle JSON Lines em stdout e o timeout absoluto do processador precisam ser verificados com testes reais.

## Rastreabilidade

Fonte integral: [`CTR-0005` — Protocolo de processador assíncrono](https://github.com/ahhvahh/a-bioma-sabia/blob/31c396fed760db93a5abcfefcfeefe6e25bf9ee3/docs/especificacao/contratos/processador-assincrono.md). As entradas, saídas, estados, erros e restrições anteriores foram preservados; referências relativas foram adaptadas à estrutura canônica.
