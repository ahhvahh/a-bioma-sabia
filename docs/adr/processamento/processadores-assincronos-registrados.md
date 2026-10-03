# Processadores assíncronos registrados e transporte de progresso

![ADR](https://img.shields.io/badge/ADR-ADR--0011-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-2-6e7781?style=flat-square)

## Contexto

O Sabiá precisa delegar trabalhos assíncronos a scripts Bash no primeiro MVP. Esses scripts podem emitir progresso, finalizar semanticamente uma requisição e produzir imagens, vídeos ou outros arquivos destinados ao cliente. Aplicações dedicadas e serviços por socket ficam como evolução futura.

## Problema

O modelo de execução orientado somente a exit code/stdout não atende processadores assíncronos. Também é necessário evitar que o canal de controle precise transportar conteúdo binário potencialmente grande.

## Restrições

- processadores precisam ser previamente cadastrados;
- comandos remotos não escolhem executável, serviço ou socket arbitrário;
- o Sabiá gera e controla `request_id`;
- progresso e finalização precisam preservar `request_id`;
- o Core não depende da Telegram Bot API;
- mídia precisa ser persistida antes de ser considerada recebida.

## Opções consideradas

### Um único protocolo com progresso e binário

Centraliza mensagens, mas mistura controle com conteúdo pesado e complica framing, memória e recovery.

### Canal de controle + canal de mídia dedicado

Mantém progresso/finalização separados do conteúdo binário. A mídia usa Unix socket local, MessagePack e persistência própria.

## Decisão

O Sabiá mantém a abstração de **processador registrado**, mas no primeiro MVP ela possui uma única implementação concreta: **script Bash previamente cadastrado**.

Cada processador referencia um `script_id` do Script Registry. Aplicações dedicadas e serviços acessíveis por socket não fazem parte do MVP e exigirão extensão explícita da arquitetura quando forem introduzidos.

O protocolo CTR-0005 usa:

- `stdin` para entregar o `request_id` UUID v4 em uma única linha;
- argumentos validados da operação via argv;
- `stdout` para eventos JSON Lines;
- `stderr` para diagnóstico.

O canal de controle usa somente:

- `loading`;
- `finally`.

Cada evento carrega `request_id` e mensagem.

Mídia não trafega pelo canal de controle.

Qualquer produtor local autorizado que possua o `request_id` pode enviar um ou vários conteúdos pelo canal de ingestão CTR-0008. O Sabiá persiste cada item e o correlaciona à mesma requisição.

`finally` encerra semanticamente o processamento normal, independentemente de quantos itens de mídia tenham sido enviados.

CTR-0002 continua válido para execução local controlada de scripts/executáveis.

## Justificativa

Separar controle e mídia mantém o protocolo assíncrono simples, permite testes independentes e garante que o conteúdo binário tenha durabilidade antes da entrega externa.

## Consequências

- a integração assíncrona reutiliza o Script Registry e o executor local;
- o stdout do script deixa de ser saída livre quando ele atua como processador assíncrono e passa a ser o canal de eventos CTR-0005;
- Media Ingest fica responsável pelo recebimento binário;
- jobs preservam `request_id` entre os dois canais;
- uma requisição pode possuir várias mídias persistidas;
- o canal de controle usa `request_id` no stdin e JSON Lines no stdout conforme CTR-0005;
- limites e autorização concreta do socket de mídia permanecem na especificação.

## Dependências

- [ADR-0003 — Core independente do Telegram](../arquitetura/core-independente-do-telegram.md)
- [ADR-0004 — Registro explícito de scripts](../execucao/registro-explicito-de-scripts.md)
- [ADR-0006 — Jobs assíncronos](jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../persistencia/estado-operacional.md)
- [ADR-0012 — Ingestão persistente de mídia por Unix socket](ingestao-midia-socket-messagepack.md)

## Critérios de validação

- scripts Bash assíncronos são previamente cadastrados e não podem ser substituídos por entrada arbitrária do Telegram;
- todo evento de controle preserva `request_id`;
- `loading` não encerra o job;
- `finally` encerra semanticamente a requisição;
- mídia não trafega no protocolo de controle;
- vários arquivos podem ser enviados pelo socket de mídia usando o mesmo `request_id`.
