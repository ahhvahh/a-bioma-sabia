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
- [CTR-0006 — Mídia temporária por requisição](../contratos/midia-temporaria.md)

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
6. a cada evento `content`, o Sabiá valida `content.path` dentro de `/tmp/sabia/media/<request_id>/`;
7. o Sabiá abre o arquivo e faz streaming ao adaptador do cliente;
8. após transmitir o último byte, o Sabiá aguarda confirmação de recebimento do Telegram;
9. somente após a confirmação, remove imediatamente o arquivo temporário;
10. o processador pode repetir `content` para cada arquivo produzido;
11. o job permanece em execução;
12. o processador envia `finally` com a mensagem final;
13. o Sabiá persiste o resultado final antes da entrega;
14. o adaptador envia a mensagem final;
15. a entrega segue a política persistente do adaptador Telegram.

## Fluxos alternativos

### loading repetido

Cada mensagem válida pode atualizar o cliente sem encerrar o job.

### finally sem arquivo

Persistir e entregar somente a mensagem final.

### content repetido

Cada evento `content` referencia um arquivo independente. Vários arquivos são enviados usando vários eventos `content` para o mesmo `request_id`.

### Falha de entrega de content

Se o streaming ou a confirmação do Telegram falhar, o arquivo não é removido e pode ser reutilizado em nova tentativa enquanto o processo atual continuar executando.

Se ocorrer shutdown/restart antes da confirmação, a limpeza da área temporária torna a mídia não recuperável; a entrega correspondente é registrada como falha e não é reenviada no próximo startup.

### content inválido

Se o caminho não pertencer ao diretório da requisição, não existir ou não puder ser lido, o arquivo não é enviado e a falha é registrada conforme CTR-0006.

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
- múltiplos arquivos podem ser transportados por vários eventos `content`;
- arquivos são transmitidos por streaming a partir da área temporária;
- arquivo só é removido após último byte transmitido e confirmação do Telegram;
- mídia não confirmada não sobrevive à limpeza de startup/shutdown;
- término de processo sem `finally` não é confundido com sucesso;
- cliente, destino e `request_id` permanecem rastreáveis.

## Implementação relacionada

Processor Registry, Processor Transport, Job Manager e adaptadores de transporte.
