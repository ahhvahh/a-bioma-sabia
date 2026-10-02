# Processadores assíncronos e transporte

![MOD](https://img.shields.io/badge/MOD-MOD--0007-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Registrar processadores assíncronos e adaptar scripts, aplicações e serviços/socket ao protocolo comum de requisição, progresso e finalização.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](../../adr/processamento/processadores-assincronos-registrados.md)
- [CTR-0002 — Execução de script](../contratos/execucao-script.md)
- [CTR-0005 — Protocolo de processador assíncrono](../contratos/processador-assincrono.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)
- [CTR-0006 — Mídia persistida](../contratos/midia-persistida.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)

## Responsabilidades

- carregar registro de processadores permitidos;
- resolver o processador de uma operação autorizada;
- gerar/adotar o `request_id` da requisição;
- selecionar o mecanismo de transporte cadastrado;
- enviar comando e argumentos ao processador;
- receber eventos de controle `loading` e `finally`;
- normalizar mecanismos concretos para CTR-0005;
- encaminhar progresso ao subsistema de jobs;
- manter o canal de controle separado da ingestão de mídia;
- permitir que processadores publiquem mídia pelo socket CTR-0008 usando o mesmo `request_id`;
- entregar a mensagem `finally` ao pipeline de resposta;
- impedir que endereço, executável ou socket arbitrário venha do usuário remoto.

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
- detalhes do mecanismo concreto ficam dentro do Processor Transport;
- binários não trafegam no protocolo de controle e nunca são enviados para logs;
- mídia produzida é enviada exclusivamente pelo socket de ingestão e persistida antes do ACK;
- um `request_id` não pode ser reaproveitado por outra execução.

## Critérios de aceite

- Bash, aplicação e socket service podem ser cadastrados sem alterar o Command Router;
- eventos de controle são normalizados para `loading | finally`;
- progresso preserva `request_id`;
- vários arquivos podem ser publicados por CTR-0008 usando o mesmo `request_id`;
- `finally` contém somente a mensagem final;
- contrato permanece `refinement` enquanto o framing do canal de controle estiver aberto.

## Implementação relacionada

Prevista para Processor Registry, Processor Transport e integração com Jobs.
