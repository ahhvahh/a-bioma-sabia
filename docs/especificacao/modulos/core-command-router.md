# Core e Command Router

![MOD](https://img.shields.io/badge/MOD-MOD--0001-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Receber comandos internos já autorizados, localizar a operação cadastrada e delegar a execução sem depender do transporte Telegram.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0003 — Core independente do Telegram](../../adr/arquitetura/core-independente-do-telegram.md)
- [CTR-0001 — Comando interno](../contratos/comando-interno.md)

## Responsabilidades

- resolver comando dentro do catálogo do cliente;
- receber `request_id`, `client_id`, `principal_id`, `command`, `arguments`, `reply_context` e `received_at` conforme CTR-0001;
- preservar `request_id`, `client_id` e `reply_context` até a produção do resultado;
- receber argumentos já tokenizados como `string[]`;
- delegar à operação correspondente a validação semântica dos argumentos;
- rejeitar comando desconhecido;
- delegar trabalho imediato ou criação de job;
- produzir resultado interno convertível pelo adaptador de transporte;
- não executar shell nem acessar diretamente Bot API.

## Entradas

Comando interno validado pelo adaptador e pela camada de autorização.

## Saídas

Envelope conforme CTR-0001 com `type: message | job | file | error`, `request_id` e `payload` compatível com o tipo.

## Interfaces e contratos

- [CTR-0001 — Comando interno](../contratos/comando-interno.md)
- [CTR-0002 — Execução de script](../contratos/execucao-script.md)
- [CTR-0003 — Job](../contratos/job.md)

## Eventos

Pode solicitar criação de job ou execução de operação cadastrada.

## Persistência

Nenhuma persistência própria foi definida para o roteador.

## Restrições

- não depender de tipos específicos da Telegram Bot API;
- não conhecer caminhos de scripts informados pelo usuário;
- não conter lógica específica de FFmpeg ou ferramenta concreta.

## Critérios de aceite

**BLOCKED para `refined`:** CTR-0001 já definiu identidade, correlação, argumentos e o envelope de resultado/erro. Permanece pendente apenas o contrato de anexos e do payload `file`.

- roteador pode ser testado com adaptadores mock;
- comando inexistente não chega a executor;
- o Router não interpreta texto bruto do transporte para obter argumentos;
- a operação correspondente valida quantidade, formato e domínio dos argumentos;
- `request_id` permanece estável durante roteamento e criação de job;
- o Router trata `reply_context.destination_id` como valor opaco;
- resultado imediato usa `message`, criação/referência assíncrona usa `job` e falhas controladas usam `error`;
- o Router não produz `file` enquanto o contrato de arquivos/anexos estiver `BLOCKED`;
- novo executor pode ser integrado sem alterar o mecanismo de identificação de comando.

## Implementação relacionada

Prevista para o pacote interno responsável por comandos/roteamento.
