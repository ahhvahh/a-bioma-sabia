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

## Responsabilidades

- carregar registro de processadores permitidos;
- resolver o processador de uma operação autorizada;
- gerar/adotar o `request_id` da requisição;
- selecionar o mecanismo de transporte cadastrado;
- enviar comando e argumentos ao processador;
- receber eventos `loading` e `finally`;
- normalizar mecanismos concretos para CTR-0005;
- encaminhar progresso ao subsistema de jobs;
- entregar resultado final, incluindo binário quando houver, ao pipeline de resposta;
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
- binários nunca são enviados para logs;
- um `request_id` não pode ser reaproveitado por outra execução.

## Critérios de aceite

- Bash, aplicação e socket service podem ser cadastrados sem alterar o Command Router;
- eventos são normalizados para `loading | finally`;
- progresso preserva `request_id`;
- resultado final pode conter mensagem e arquivo;
- contrato permanece `refinement` enquanto framing, mídia e limites estiverem abertos.

## Implementação relacionada

Prevista para Processor Registry, Processor Transport e integração com Jobs.
