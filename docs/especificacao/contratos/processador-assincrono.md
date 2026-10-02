# Protocolo de processador assíncrono

![CTR](https://img.shields.io/badge/CTR-CTR--0005-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir o protocolo interno comum entre o Sabiá e processadores assíncronos registrados, independentemente de o processador ser um script Bash, aplicação/executável ou serviço acessível por socket.

## Dependências

- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](../../adr/processamento/processadores-assincronos-registrados.md)
- [CTR-0001 — Comando interno](comando-interno.md)
- [CTR-0003 — Job](job.md)
- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [CTR-0006 — Mídia temporária por requisição](midia-temporaria.md)

## Tipo

`mensagem`

## Entrada do Sabiá para o processador

Toda execução assíncrona recebe, no mínimo:

- `request_id: string` — identificador gerado e controlado pelo Sabiá;
- `command: string` — operação lógica resolvida pelo registro;
- `arguments: string[]` — argumentos já validados pela operação.

Quando a operação aceitar mídia de entrada, a requisição também pode transportar anexos conforme CTR-0001. O objeto binário de entrada permanece `BLOCKED` até o contrato de mídia ser refinado.

O processador não gera nem substitui o `request_id`.

## Eventos do processador para o Sabiá

Cada evento contém obrigatoriamente:

- `request_id: string`;
- `status: loading | content | finally`.

`message` é obrigatório em `loading` e `finally`. Em `content`, o payload é o objeto de mídia definido em CTR-0006.

### loading

`loading` representa atualização intermediária.

Regras:

- pode ocorrer zero ou mais vezes;
- não encerra o job;
- não altera o `request_id`;
- a mensagem pode ser encaminhada ao cliente correlacionado.

### content

`content` anuncia um arquivo produzido pelo processador.

O objeto contém:

- `request_id: string`;
- `status: content`;
- `content.name: string`;
- `content.path: string`.

Cada evento referencia um único arquivo. Uma requisição pode publicar zero ou mais eventos `content`.

O arquivo precisa estar dentro de `/tmp/sabia/media/<request_id>/` e segue CTR-0006.

`content` não encerra o job.

### finally

`finally` representa a finalização semântica da requisição.

O objeto final contém:

- `request_id: string`;
- `status: finally`;
- `message: string`.

`finally` não transporta arquivo nem conteúdo binário.

Após um `finally` válido ser aceito, a requisição é terminal e novos eventos para o mesmo `request_id` não podem alterar seu resultado.

## Relação com jobs

O `request_id` é a correlação entre comando, job, processador, mensagens de progresso e resultado final.

`loading` representa progresso do job em execução, mas não cria novo estado terminal em CTR-0003.

`content` representa conteúdo de saída e pode ocorrer várias vezes sem concluir o job.

`finally` permite concluir o processamento do job e preparar a mensagem final para entrega.

## Saída para o cliente

Mensagens `loading` podem ser convertidas pelo adaptador em atualização ao cliente Telegram correspondente.

Eventos `content` são validados conforme CTR-0006 e o arquivo é transmitido por streaming ao cliente correlacionado.

O evento `finally` deve produzir a mensagem final persistida.

## Erros

Condições de erro incluem:

- `unknown_request` — `request_id` não corresponde a execução ativa;
- `invalid_status` — status diferente de `loading`, `content` ou `finally`;
- `invalid_content_payload` — objeto `content` ausente ou inválido;
- `processor_timeout` — nenhum `finally` foi recebido dentro do limite operacional;
- `transport_failure` — falha no mecanismo concreto entre Sabiá e processador.

## Regras e restrições

- somente processadores cadastrados podem receber requisições;
- eventos não podem acessar diretamente o adaptador Telegram;
- toda correlação usa `request_id`;
- `loading` não finaliza processamento;
- `content` não finaliza processamento e pode ocorrer múltiplas vezes;
- `finally` é terminal;
- conteúdo binário não faz parte do protocolo de controle e não deve ser escrito em logs;
- stdout/stderr de CTR-0002 não substituem este protocolo assíncrono.

## Compatibilidade

Scripts, aplicações e serviços/socket podem usar mecanismos concretos diferentes, desde que o Processor Transport normalize todos para este contrato.

## BLOCKED

Ainda precisam ser definidos antes de `refined`:

- representação/framing das mensagens para cada transporte concreto;
- framing das mensagens para cada transporte concreto;
- política de limpeza/recovery de mídia temporária de CTR-0006;
- determinação normativa do tipo de mídia;
- limites máximos de arquivo.

## Critérios de aceite

- toda execução recebe `request_id` gerado pelo Sabiá;
- todo evento devolve o mesmo `request_id`;
- `loading` pode ser encaminhado ao Telegram sem encerrar o job;
- `finally` encerra semanticamente o processamento;
- `content` referencia exatamente um arquivo por evento;
- múltiplos arquivos usam múltiplos eventos `content`;
- `finally` não carrega binário;
- scripts, aplicações e socket services usam o mesmo modelo conceitual;
- detalhes bloqueados acima precisam ser refinados antes da implementação.
