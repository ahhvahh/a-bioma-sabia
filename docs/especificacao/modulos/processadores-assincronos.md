# Processadores assíncronos e transporte

![MOD](https://img.shields.io/badge/MOD-MOD--0007-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Registrar scripts Bash assíncronos e adaptar sua execução ao protocolo comum de progresso e finalização do MVP.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](../../adr/processamento/processadores-assincronos-registrados.md)
- [CTR-0002 — Execução de script](../contratos/execucao-script.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)
- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)

## Responsabilidades

- carregar registro de processadores permitidos;
- resolver o processador de uma operação autorizada;
- gerar/adotar o `request_id` da requisição;
- resolver o `script_id` cadastrado no Script Registry;
- iniciar o script com os argumentos validados da operação;
- escrever o `request_id` no stdin do processo;
- receber eventos de controle `loading` e `finally`;
- interpretar o stdout do script como JSON Lines UTF-8 conforme CTR-0005;
- encaminhar progresso ao subsistema de jobs;
- manter o canal de controle separado da ingestão de mídia;
- permitir que processadores publiquem mídia pelo socket CTR-0008 usando o mesmo `request_id`;
- entregar a mensagem `finally` ao pipeline de resposta;
- impedir que identificador, caminho ou executável arbitrário venha do usuário remoto.

## Entradas

Operação assíncrona já autorizada e resolvida pelo Command Router.

## Saídas

Eventos normalizados conforme CTR-0005.

## Interfaces e contratos

- [CTR-0002 — Execução de script](../contratos/execucao-script.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)

## Persistência

A definição do processador vem da configuração.

Estado de requisição, job, progresso relevante, resultado final e entrega pertencem ao estado operacional persistente.

## Restrições

- nenhuma mensagem Telegram vira caminho, socket ou executável;
- no MVP existe apenas execução de script Bash; stdin carrega `request_id`, stdout carrega eventos JSON Lines e stderr permanece diagnóstico;
- binários não trafegam no protocolo de controle e nunca são enviados para logs;
- mídia produzida é enviada exclusivamente pelo socket de ingestão e persistida antes do ACK;
- um `request_id` não pode ser reaproveitado por outra execução.

## Critérios de aceite

- somente scripts Bash previamente cadastrados podem atuar como processadores assíncronos no MVP;
- cada processador referencia um `script_id` existente no Script Registry;
- eventos de controle são normalizados para `loading | finally`;
- progresso preserva `request_id`;
- vários arquivos podem ser publicados por CTR-0008 usando o mesmo `request_id`;
- `finally` contém somente a mensagem final;
- `request_id` é entregue por stdin e JSON Lines é o formato único do stdout de controle;
- aplicações dedicadas e serviços por socket ficam fora do MVP;
- timeout sem `finally` segue a política absoluta de 2 horas de CTR-0005.

## Implementação relacionada

Prevista para Processor Registry, Processor Transport e integração com Jobs.
