<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# REF-003 — Requisitos de testes

**ID:** `REF-003` · **Origem histórica:** `REQ-0002`  
**Estado original:** `refined` · **Implementação:** não verificada nesta branch  
**Classificação:** revisão documental; itens em `refinement` não foram aprovados por esta migração.

## Objetivo

Definir a cobertura mínima de testes necessária para o primeiro MVP.

## Dependências

- [ARQ-002 (DSG-0002) — Componentes do Sabiá Core](../arquitetura/ARQ-002-componentes-core.md)
- [REQ-0001 — Escopo do primeiro MVP](REF-002-escopo-primeiro-mvp.md)

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
- abertura fracionada rejeita `total_bytes` acima de 50 MB antes de criar `media_id`;
- abertura de upload fracionado retorna `media_id` e `next_sequence_id = 1`;
- sequência pulada retorna `sequence_gap` com valor esperado;
- sequência já recebida retorna `sequence_already_received` sem duplicar persistência;
- chunks persistidos podem ser lidos em ordem sem montar o arquivo inteiro em memória;
- `received_bytes == total_bytes` marca mídia fracionada como completa;
- chunk que ultrapassaria `total_bytes` é recusado;
- limpeza de payload não executa com transmissão `pending` ou `transmitting`;
- payload `delivered` é removível sem apagar metadados de rastreabilidade.

## Plano de testes Telegram 2026-10-21

Testar isoladamente (sem API real) os grupos: T01 inbox/commit/cursor/crash/replay; T02 menu/callback/voltar/início/autorização; T03 formatos de mídia e limites até 50 MB; T04 backoff/429/retry/restart; T05 isolamento entre três clientes. Fixtures, procedimentos e critérios de aceite: [testes/telegram/README.md](../testes/telegram/README.md). Necessários mock Bot API HTTP, PostgreSQL temporário e testes de contrato automatizados ao implementar `/src`.

## Pendências e integração

A cobertura proposta não comprova execução. O [caderno Telegram](../testes/telegram/README.md) referencia grupos T01–T05, mas a existência de fixtures e testes automatizados deve ser verificada antes de declarar conformidade. Ver [escopo](REF-002-escopo-primeiro-mvp.md).

## Rastreabilidade

Documento original preservado: [REQ-0002 — Requisitos de testes](../especificacao/requisitos/testes.md).
