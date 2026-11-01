# OBJ-009 — Permitir assinaturas de tópicos e notificações por evento no Telegram

![Tipo: OBJ](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-type-obj.svg) ![Status: refinement](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-version-v1.svg)

**ID:** OBJ-009

## Objetivo

Permitir que usuários autorizados assinem e cancelem assinaturas de tópicos apresentados nos clientes Telegram do Sabiá, recebendo notificações sempre que aplicações locais autorizadas publiquem eventos nesses tópicos.

## Motivação

Permitir monitoramento e avisos contínuos de serviços locais sem obrigar a pessoa usuária a solicitar atualizações repetidamente e sem obrigar os produtores a conhecer credenciais ou destinatários do Telegram.

## Resultado esperado

Cada tópico publicado por uma aplicação autorizada pode ser apresentado a usuários com acesso permitido. Ao assiná-lo, o usuário passa a receber as novas publicações destinadas àquele tópico. Inscrições, publicações, sequências de eventos e entregas precisam manter consistência durante reinicializações do Sabiá.

## Critérios verificáveis

- Uma aplicação autorizada disponibiliza um tópico e pode publicar eventos sem acessar diretamente contatos ou tokens do Telegram.
- Um usuário autorizado visualiza os tópicos permitidos, assina e cancela a assinatura pelo Telegram.
- Cada publicação aceita gera notificações para os destinatários inscritos aplicáveis ao momento do evento.
- Os eventos destinados a um mesmo assinante e tópico respeitam a ordem de aceitação das publicações, inclusive após reinicialização.
- Eventos e entregas são preservados para recuperação de falhas; as entregas concluídas são rastreáveis.
- Identidades não autorizadas não podem publicar em nome de terceiros nem acessar tópicos restritos.
- A confirmação de aceite de uma publicação não deve ser confundida com confirmação de leitura pelo destinatário.

## Limites

Esta decisão se refere a assinaturas **acionadas por eventos publicados**. Mailings e geração de conteúdo por periodicidade continuam sujeitos a definição de escopo própria. Nenhuma assinatura implica acesso a operações remotas, executáveis ou dados de outro cliente Telegram. O Charter não define formatos de transporte, tipo de socket, tabelas ou estratégias de implementação.

## Relações

- Charter vigente: [Project Charter](../README.md).
- Exclusão revogada: [NOBJ-006](../non-objectives/NOBJ-006-assinaturas-excluidas-cancelado.md).
- Scope já existente, somente como referência: [IN-010 — Contatos](../../scope/in-scope/IN-010-registro-persistente-de-contatos.md), [IN-011 — Catálogo](../../scope/in-scope/IN-011-catalogo-persistente-de-acoes-e-menu.md), [IN-013 — Assinaturas periódicas em backlog](../../scope/in-scope/IN-013-assinaturas-periodicas.md).
- **PENDING — Scope:** contrato próprio para publicação e assinaturas por evento, integridade/ordenação da entrega, autorização, estruturas de dados e integração local.
- **PENDING — Implementation:** nova biblioteca LIB-007 e adequações do SVC-001 somente após os gates de origem.

**Evidência:** decisão expressa do responsável no chat de 21/10/2026; ainda sem contratos de Scope refinados ou código implementado.
