# Core independente do Telegram

![ADR](https://img.shields.io/badge/ADR-ADR--0003-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

Telegram é o canal inicial, mas o projeto prevê integrações futuras por Unix Socket, TCP, HTTP/REST, gRPC, systemd, Docker e aplicações Bioma.

## Problema

Evitar que regras de execução, jobs e monitoramento fiquem acopladas à Telegram Bot API.

## Restrições

- a camada Telegram deve apenas traduzir eventos e respostas;
- novos executores não devem exigir alteração do Command Router;
- o núcleo não deve depender de FFmpeg ou ferramenta específica.

## Opções consideradas

### Core orientado à Telegram Bot API

Simplifica o início, mas acopla domínio e transporte.

### Core com contratos internos e adaptadores

Mantém transporte e executores substituíveis.

## Decisão

Separar Telegram, roteamento de comandos e executores por interfaces internas. O Core trabalha com comandos e resultados internos; Telegram é um adaptador de entrada/saída.

## Justificativa

Permite expansão dos canais e executores sem alterar a responsabilidade do roteador.

## Consequências

- objetos específicos da Bot API não devem atravessar indiscriminadamente o Core;
- scripts, ferramentas e futuras integrações são acessados por executores;
- respostas precisam ser convertíveis pelo adaptador Telegram.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](../telegram/multiplos-clientes-isolados.md)

## Critérios de validação

- Command Router pode ser testado sem Telegram real;
- um executor futuro pode ser adicionado sem mudar a regra de roteamento;
- testes do Core usam mocks das interfaces externas.
