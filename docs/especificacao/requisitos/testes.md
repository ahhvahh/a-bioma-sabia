# Requisitos de testes

**ID:** REQ-0002  
**Status:** refined

## Objetivo

Definir a cobertura mínima de testes necessária para o primeiro MVP.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [REQ-0001 — Escopo do primeiro MVP](mvp.md)

## Cobertura mínima

Devem existir testes para:
- Command Router;
- Script Registry;
- autorização;
- executor;
- timeout;
- Job Manager;
- Scheduler;
- mudança de estado dos alertas;
- configuração.

## Isolamento externo

Nenhum teste deve depender da Telegram API real.

Interfaces externas devem permitir mocks/fakes para validar o Core de forma determinística.

## Critérios de aceite

- suíte principal executa sem conectividade com Telegram;
- autorização possui cenários permitido e negado;
- executor cobre sucesso, erro e timeout;
- scheduler cobre disparo devido;
- alertas cobrem mudança, repetição e recuperação;
- configuração cobre entrada válida e inválida.
