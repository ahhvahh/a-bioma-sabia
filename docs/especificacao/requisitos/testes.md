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
- configuração;
- ingestão simples de mídia;
- ingestão fracionada de mídia;
- persistência e ordenação de chunks;
- recovery de transmissões de mídia.

## Isolamento externo

Nenhum teste deve depender da Telegram API real.

Interfaces externas devem permitir mocks/fakes para validar o Core de forma determinística.

## Critérios de aceite

- suíte principal executa sem conectividade com Telegram;
- autorização possui cenários permitido e negado;
- executor cobre sucesso, erro e timeout;
- scheduler cobre disparo devido;
- alertas cobrem mudança, repetição e recuperação;
- configuração cobre entrada válida e inválida;
- canal simples rejeita payload acima de 20 MB;
- canal fracionado rejeita chunk acima de 5 MB;
- abertura de upload fracionado retorna `media_id` e `next_sequence_id = 1`;
- sequência pulada retorna `sequence_gap` com valor esperado;
- sequência já recebida retorna `sequence_already_received` sem duplicar persistência;
- chunks persistidos podem ser lidos em ordem sem montar o arquivo inteiro em memória.
