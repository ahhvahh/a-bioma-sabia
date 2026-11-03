# Comando interno

![CTR](https://img.shields.io/badge/CTR-CTR--0001-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Definir a fronteira entre adaptadores de entrada e o Command Router sem transportar dependência direta da Telegram Bot API para o Core.

## Dependências

- [MOD-0001 — Core e Command Router](../modulos/core-command-router.md)
- [MOD-0002 — Adaptador Telegram](../modulos/telegram.md)
- [ADR-0003 — Core independente do Telegram](../../adr/arquitetura/core-independente-do-telegram.md)
- [CTR-0005 — Protocolo de processador assíncrono](processador-assincrono.md)
- [CTR-0006 — Mídia persistida](midia-persistida.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](ingestao-midia-messagepack.md)

## Tipo

`interface`

## Entrada

O comando interno usa o seguinte schema mínimo:

- `request_id: string` — identificador único da requisição dentro do Sabiá;
- `client_id: string` — cliente lógico que recebeu a solicitação;
- `principal_id: string` — identidade já autorizada, normalizada pelo adaptador;
- `command: string` — identificador lógico do comando;
- `arguments: string[]` — argumentos já tokenizados e ainda não validados semanticamente;
- `reply_context.transport: string` — identificador lógico do transporte de origem;
- `reply_context.destination_id: string` — destino opaco ao Core usado pelo adaptador para responder;
- `received_at: timestamp` — instante em que a requisição foi aceita pelo adaptador;
- `attachments: attachment[]` — opcional, somente quando a operação suportar anexos.

### Identidade e correlação

`request_id` é um UUID v4 gerado pelo Sabiá na entrada e permanece estável durante roteamento, criação de job, persistência, auditoria e entrega da resposta. Em JSON/MessagePack ele é transportado como string UUID canônica com hífens.

`client_id` identifica qual cliente lógico recebeu a solicitação e deve ser usado para selecionar o adaptador/credencial correto na resposta.

`principal_id` representa a identidade já submetida à autorização. Para Telegram, o adaptador normaliza o Telegram User ID para string antes de construir o comando interno.

`reply_context` não carrega objetos do SDK ou da Bot API. Ele contém somente:

- `transport`: nome lógico do transporte, como `telegram`;
- `destination_id`: identificador do destino no transporte, tratado como valor opaco pelo Core.

Para Telegram, `destination_id` corresponde ao chat de resposta normalizado como string. O Core não interpreta seu formato.

`received_at` deve representar um instante absoluto; a representação concreta em memória pode variar, mas serializações persistidas devem preservar data/hora e fuso ou normalização equivalente em UTC.

### Argumentos

Os argumentos entram no Core como:

`arguments: string[]`

Regras:

- o adaptador de entrada é responsável por transformar a representação recebida no transporte em uma lista ordenada de strings;
- ausência de argumentos é representada por lista vazia;
- o adaptador não aplica validação semântica específica da operação;
- quantidade, formato, domínio, obrigatoriedade e significado de cada argumento pertencem à operação resolvida pelo Command Router;
- o Core não recebe texto bruto do Telegram para interpretar a sintaxe do comando;
- nenhum objeto específico da Telegram Bot API pode ser carregado em `arguments`.

### Anexos

`attachments` é opcional e cada item referencia mídia persistida conforme CTR-0006:

- `media_id: integer`;
- `name: string`;
- `content_type: string | null`.

O adaptador ou produtor persiste a mídia antes de disponibilizá-la ao Core. O Core não recebe binário no envelope.

### Regra de `content_type`

`content_type` é metadado declarado, não uma fonte de autorização ou segurança.

Regras normativas:

- `null` é válido quando o tipo não for conhecido;
- string vazia ou composta somente por espaços é normalizada para `null`;
- valor não nulo deve possuir sintaxe válida de media type, como `image/jpeg`, `video/mp4` ou `application/pdf`;
- o valor persistido e propagado é normalizado para minúsculas;
- o Sabiá não infere `content_type` pela extensão de `name`;
- o Sabiá não inspeciona os bytes para detectar o tipo no MVP;
- media type válido, porém não reconhecido pelo transporte, continua válido como metadado;
- valor sintaticamente inválido produz erro controlado `invalid_content_type`.

## Saída

Toda resposta do Core usa um envelope discriminado:

- `type: message | job | file | error`;
- `request_id: string`;
- `payload`: conteúdo específico do tipo.

`request_id` deve ser o mesmo da requisição de origem.

### message

Representa resposta textual imediata.

Payload mínimo:

- `text: string`.

### job

Representa criação ou referência de job assíncrono.

Payload mínimo:

- `job_id: integer`.

O ciclo de vida do job segue [CTR-0003 — Job](job.md).

### file

Representa mídia persistida pronta para entrega.

Payload:

- `media_id: integer`;
- `name: string`;
- `content_type: string | null`.

Processadores assíncronos publicam vários arquivos realizando vários uploads CTR-0008 com o mesmo `request_id`.

### error

Representa erro controlado produzido pelo Core ou por uma operação.

Payload obrigatório:

- `code: string` — código estável e apropriado para tratamento programático;
- `message: string` — mensagem controlada e apropriada para apresentação ao solicitante;
- `retryable: boolean` — indica se repetir a mesma operação pode ser apropriado sem alteração da solicitação.

O adaptador pode adaptar a apresentação do erro ao transporte, mas não deve reinterpretar o significado de `code` ou `retryable`.

## Erros

Erros previstos incluem, no mínimo:

- comando desconhecido;
- argumentos inválidos;
- operação não disponível para o cliente;
- falha do executor.

Todos devem ser retornados usando `type: error` e o payload normativo definido acima.

Códigos específicos por operação pertencem ao contrato da própria operação.

## Regras e restrições

- não expor objetos da Bot API como contrato do domínio;
- autorização ocorre antes da execução;
- comando não carrega caminho de script arbitrário;
- tokenização é responsabilidade do adaptador;
- validação semântica dos argumentos é responsabilidade da operação correspondente;
- `request_id` não muda durante o processamento da mesma requisição;
- resposta deve preservar `request_id`, `client_id` e `reply_context` na correlação operacional, mesmo que o envelope retornado ao adaptador carregue diretamente apenas `request_id`;
- o Core trata `destination_id` como opaco;
- exatamente um valor de `type` é válido por resultado;
- `payload` deve ser compatível com o `type` correspondente.

## Compatibilidade

Novos transportes devem conseguir produzir o mesmo comando conceitual, normalizar sua identidade para `principal_id`, fornecer `reply_context` e entregar argumentos como `string[]`, independentemente da sintaxe original do transporte.

Da mesma forma, adaptadores futuros devem conseguir converter o envelope de resultado sem exigir que o Core conheça detalhes do transporte.

## Critérios de aceite

- o Router funciona em teste sem Telegram;
- argumentos chegam ao Core como `string[]` já tokenizado;
- adaptadores não precisam conhecer as regras semânticas específicas de cada operação;
- operações validam seus próprios argumentos;
- uma requisição possui `request_id` UUID v4 estável ponta a ponta;
- o Core não depende de Telegram User ID ou Chat ID como tipos nativos;
- a resposta pode ser roteada por `client_id` + `reply_context`;
- resultados usam `type: message | job | file | error`;
- todo resultado preserva `request_id`;
- `message` contém `text`;
- `job` contém `job_id`;
- `error` contém `code`, `message` e `retryable`;
- referências de mídia de entrada e saída usam `media_id` conforme CTR-0006;
- `content_type` segue a regra normativa deste contrato; retenção do payload é tratada por CTR-0006/FLW-0008 e o limite fracionado pertence a CTR-0009.
