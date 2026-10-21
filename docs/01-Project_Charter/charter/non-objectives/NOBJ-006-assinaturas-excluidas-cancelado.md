# NOBJ-006 — Exclusão de assinaturas de usuários (cancelada)

![Tipo: NOBJ](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-type-nobj.svg) ![Status: cancelled](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-status-cancelled.svg) ![Versão: v1](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-version-v1.svg)

**ID:** NOBJ-006

## Exclusão anterior

No primeiro MVP, não entregar o mecanismo completo de assinaturas criadas pelo usuário e mailings periódicos, como notícias e conteúdo recorrente.

## Justificativa histórica

A capacidade de assinaturas foi anteriormente adiada para evolução posterior, com vínculo a [IN-013](../../scope/in-scope/IN-013-assinaturas-periodicas.md).

## Cancelamento da exclusão

**Decisão em 21/10/2026:** o projeto passa a oferecer **assinatura de tópicos por usuários autorizados do Telegram**, com recebimento de notificações a cada evento publicado por aplicação local autorizada. Essa parte da exclusão anterior deixa de ser válida para o produto.

O recurso agora é objetivo ativo em [OBJ-009](../objectives/OBJ-009-assinatura-e-entrega-de-eventos-telegram.md).

## Delimitação

Cancelamento da exclusão anterior **não transforma automaticamente** os mailings periódicos do item IN-013 em entrega ativa. A funcionalidade periódica continua aguardando definição específica; a entrega de eventos assinados é a mudança aprovada.

## Rastreabilidade e próximos gates

- **DOCUMENTED:** exclusão histórica NOBJ-006 cancelada por decisão do usuário.
- **PENDING — Scope:** rever OUT-009 e o relacionamento de IN-013 com a exclusão cancelada; adicionar a capacidade de assinatura por evento e seus contratos funcionais.
- **PENDING — Implementation:** descrever LIB-007 e ampliar SVC-001 após os gates documentais.
