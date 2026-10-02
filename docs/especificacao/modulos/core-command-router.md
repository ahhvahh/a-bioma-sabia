# Core e Command Router

![MOD](https://img.shields.io/badge/MOD-MOD--0001-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Receber comandos internos já autorizados, localizar a operação cadastrada e delegar a execução sem depender do transporte Telegram.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0003 — Core independente do Telegram](../../adr/arquitetura/core-independente-do-telegram.md)
- [CTR-0001 — Comando interno](../contratos/comando-interno.md)

## Responsabilidades

- resolver comando dentro do catálogo do cliente;
- rejeitar comando desconhecido;
- delegar trabalho imediato ou criação de job;
- produzir resultado interno convertível pelo adaptador de transporte;
- não executar shell nem acessar diretamente Bot API.

## Entradas

Comando interno validado pelo adaptador e pela camada de autorização.

## Saídas

Resultado interno imediato, erro de roteamento ou referência para job criado.

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

- roteador pode ser testado com adaptadores mock;
- comando inexistente não chega a executor;
- novo executor pode ser integrado sem alterar o mecanismo de identificação de comando.

## Implementação relacionada

Prevista para o pacote interno responsável por comandos/roteamento.
