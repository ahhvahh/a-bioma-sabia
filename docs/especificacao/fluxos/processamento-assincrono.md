# Processamento assíncrono por processador registrado

![FLW](https://img.shields.io/badge/FLW-FLW--0005-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Executar um comando em processador assíncrono registrado, encaminhar progresso ao cliente e entregar a resposta final, incluindo arquivo quando houver.

## Dependências

- [MOD-0007 — Processadores assíncronos e transporte](../modulos/processadores-assincronos.md)
- [MOD-0004 — Jobs](../modulos/jobs.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)
- [CTR-0001 — Comando interno](../contratos/comando-interno.md)

## Gatilho

Command Router resolve uma operação cadastrada como processamento assíncrono.

## Pré-condições

- operação autorizada;
- processador cadastrado;
- `request_id`, `client_id` e `reply_context` disponíveis;
- capacidade de job disponível.

## Fluxo principal

1. o Sabiá cria/persiste o job e associa o `request_id`;
2. o Processor Registry resolve o processador cadastrado;
3. o Processor Transport envia `request_id`, comando e argumentos;
4. o processador inicia o trabalho;
5. a cada evento `loading`, o Sabiá valida o `request_id` e encaminha `message` ao cliente correlacionado;
6. o job permanece em execução;
7. o processador envia `finally` com a mensagem final;
8. quando existir arquivo, `finally` também contém `file_type` e `binary`;
9. o Sabiá persiste o resultado final antes da entrega;
10. o adaptador do cliente envia mensagem e, quando houver, o arquivo;
11. a entrega segue a política persistente do adaptador Telegram.

## Fluxos alternativos

### loading repetido

Cada mensagem válida pode atualizar o cliente sem encerrar o job.

### finally sem arquivo

Persistir e entregar somente a mensagem final.

### finally com arquivo

Persistir metadados e conteúdo/referência conforme o contrato de mídia ainda a ser refinado e entregar o arquivo pelo transporte de origem.

### Processo termina sem finally

A requisição não é considerada concluída com sucesso apenas pelo término do processo. O tratamento final depende da política de timeout/falha definida para CTR-0005.

## Falhas e tratamento

- `request_id` desconhecido: rejeitar evento e auditar;
- payload inválido: não concluir o job;
- falha do Processor Transport: registrar falha do processamento;
- timeout sem `finally`: finalizar conforme política de timeout do job;
- falha de entrega ao Telegram não altera o resultado do processamento e segue a fila de entrega.

## Resultado

O job possui resultado final correlacionado ao `request_id` e a resposta permanece persistida até ser entregue.

## Critérios de aceite

- progresso `loading` chega ao cliente correto;
- progresso não finaliza o job;
- `finally` é necessário para conclusão semântica normal;
- arquivo final pode ser transportado quando o contrato binário estiver refinado;
- término de processo sem `finally` não é confundido com sucesso;
- cliente, destino e `request_id` permanecem rastreáveis.

## Implementação relacionada

Processor Registry, Processor Transport, Job Manager e adaptadores de transporte.
