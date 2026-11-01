# OBJ-009 — Assinaturas de tópicos e notificações por evento

![Tipo: OBJ](../assets/documentation-style/badges/badge-type-obj.svg) ![Status: refinement](../assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](../assets/documentation-style/badges/badge-version-v1.svg)

## Objetivo

Usuários autorizados podem assinar ou cancelar tópicos e receber eventos publicados por aplicações locais autorizadas.

## Resultado esperado e verificação

As condições verificáveis aprovadas são:

- Uma aplicação local autorizada disponibiliza um tópico e publica eventos sem acessar contatos ou tokens Telegram.
- O usuário autorizado visualiza os tópicos permitidos, assina e cancela a assinatura pelo Telegram.
- Cada publicação aceita gera notificação para os destinatários inscritos aplicáveis no momento de publicação.
- O mesmo assinante e tópico recebem eventos na ordem de aceitação, mesmo após reinício.
- Eventos e entregas são preservados para recuperação de falhas e rastreabilidade das entregas concluídas.
- Identidades não autorizadas não publicam em nome de terceiros nem acessam tópicos restritos.
- O aceite da publicação não equivale a confirmação de leitura pelo destinatário.

## Rastreabilidade

- PCH: [Contexto e atores](../project-context/PCH-001-contexto-e-atores-do-sabia.md)
- PENDING — Scope canônico: PENDING — Scope: contrato próprio para tópicos/eventos; IN-013 trata separadamente assinaturas periódicas.
- Situação de revisão: demanda em aberto; o status permanece em `refinement`.

## Delimitação preservada

Assinatura por **evento publicado** é ativa por decisão de 21/10/2026. Mailing periódico permanece em backlog e não é incluído implicitamente; a exclusão histórica consta em [NOBJ-006](../non-objectives/NOBJ-006-assinaturas-excluidas-cancelado.md).
