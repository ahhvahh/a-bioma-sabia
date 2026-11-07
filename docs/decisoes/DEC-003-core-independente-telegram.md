<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# DEC-003 — Core independente do Telegram

**ID:** `DEC-003`  
**ID histórico:** `ADR-0003`  
**Estado documental de origem:** `refined`  
**Versão histórica registrada:** `1`  
**Implementação:** não verificada nesta revisão

## Decisão

Separar Telegram, roteamento de comandos e executores por interfaces internas. O Core trabalha com comandos e resultados internos; Telegram é um adaptador de entrada/saída.

## Contexto e problema

Telegram é o canal inicial, mas o projeto prevê integrações futuras por Unix Socket, TCP, HTTP/REST, gRPC, systemd, Docker e aplicações Bioma.

Evitar que regras de execução, jobs e monitoramento fiquem acopladas à Telegram Bot API.

## Restrições

- a camada Telegram deve apenas traduzir eventos e respostas;
- novos executores não devem exigir alteração do Command Router;
- o núcleo não deve depender de FFmpeg ou ferramenta específica.

## Alternativas consideradas

### Core orientado à Telegram Bot API

Simplifica o início, mas acopla domínio e transporte.

### Core com contratos internos e adaptadores

Mantém transporte e executores substituíveis.

## Justificativa

Permite expansão dos canais e executores sem alterar a responsabilidade do roteador.

## Consequências

- objetos específicos da Bot API não devem atravessar indiscriminadamente o Core;
- scripts, ferramentas e futuras integrações são acessados por executores;
- respostas precisam ser convertíveis pelo adaptador Telegram.

## Critérios de validação

- Command Router pode ser testado sem Telegram real;
- um executor futuro pode ser adicionado sem mudar a regra de roteamento;
- testes do Core usam mocks das interfaces externas.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](DEC-002-clientes-telegram-isolados.md)

## Limitações da revisão

TCP, HTTP, gRPC e outros canais são possibilidades futuras, não interfaces implementadas ou aprovadas para o primeiro MVP.

A revisão documental não demonstra implementação, execução de testes ou aprovação de novas regras.

## Fonte e rastreabilidade

Documento histórico preservado: [`ADR-0003` original](https://github.com/ahhvahh/a-bioma-sabia/blob/f898239c4519b1c8fd21327adf7a8407a1bbeaf7/docs/adr/arquitetura/core-independente-do-telegram.md). O identificador e a versão original foram mantidos para rastrear a decisão, sem renumerar outros ADRs.
