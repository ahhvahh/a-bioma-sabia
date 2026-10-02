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
- `status: loading | finally`;
- `message: string`.

### loading

`loading` representa atualização intermediária.

Regras:

- pode ocorrer zero ou mais vezes;
- não encerra o job;
- não altera o `request_id`;
- a mensagem pode ser encaminhada ao cliente correlacionado.

### finally

`finally` representa a finalização semântica da requisição.

O objeto final contém:

- `request_id: string`;
- `status: finally`;
- `message: string`;
- `file_type: string | null`;
- `binary: bytes | null`.

Quando não houver arquivo, `file_type` e `binary` são nulos.

Quando houver arquivo, ambos precisam estar presentes.

Após um `finally` válido ser aceito, a requisição é terminal e novos eventos para o mesmo `request_id` não podem alterar seu resultado.

## Relação com jobs

O `request_id` é a correlação entre comando, job, processador, mensagens de progresso e resultado final.

`loading` representa progresso do job em execução, mas não cria novo estado terminal em CTR-0003.

`finally` permite concluir o processamento do job e preparar sua resposta para entrega.

## Saída para o cliente

Mensagens `loading` podem ser convertidas pelo adaptador em atualização ao cliente Telegram correspondente.

O evento `finally` deve produzir uma resposta final persistida. Quando houver `binary`, a camada de transporte de mídia deve convertê-lo em resultado `file` de CTR-0001 e enviá-lo pelo adaptador correspondente.

## Erros

Condições de erro incluem:

- `unknown_request` — `request_id` não corresponde a execução ativa;
- `invalid_status` — status diferente de `loading` ou `finally`;
- `invalid_final_payload` — somente um de `file_type` ou `binary` foi informado;
- `processor_timeout` — nenhum `finally` foi recebido dentro do limite operacional;
- `transport_failure` — falha no mecanismo concreto entre Sabiá e processador.

## Regras e restrições

- somente processadores cadastrados podem receber requisições;
- eventos não podem acessar diretamente o adaptador Telegram;
- toda correlação usa `request_id`;
- `loading` não finaliza processamento;
- `finally` é terminal;
- resultado binário não deve ser escrito em logs;
- stdout/stderr de CTR-0002 não substituem este protocolo assíncrono.

## Compatibilidade

Scripts, aplicações e serviços/socket podem usar mecanismos concretos diferentes, desde que o Processor Transport normalize todos para este contrato.

## BLOCKED

Ainda precisam ser definidos antes de `refined`:

- representação/framing das mensagens para cada transporte concreto;
- semântica normativa de `file_type` — MIME type, extensão, enumeração ou combinação;
- limite máximo do `binary`;
- estratégia para binários grandes, especialmente vídeo;
- objeto de mídia de entrada e sua relação com `attachments` de CTR-0001.

## Critérios de aceite

- toda execução recebe `request_id` gerado pelo Sabiá;
- todo evento devolve o mesmo `request_id`;
- `loading` pode ser encaminhado ao Telegram sem encerrar o job;
- `finally` encerra semanticamente o processamento;
- `finally` sem arquivo aceita `file_type = null` e `binary = null`;
- `finally` com arquivo exige tipo e conteúdo binário;
- scripts, aplicações e socket services usam o mesmo modelo conceitual;
- detalhes bloqueados acima precisam ser refinados antes da implementação.
