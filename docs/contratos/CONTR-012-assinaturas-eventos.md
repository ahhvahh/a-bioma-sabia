<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# CONTR-012 — Requisitos de assinatura e entrega por eventos

**Situação:** `refinement`; contrato funcional **incompleto e não implementado**. Registra o objetivo ativo `OBJ-009` citado no consolidado histórico; não inventa operações técnicas, payloads nem tabelas.

## Objetivo e distinções

Permitir que **usuários autorizados assinem ou cancelem assinaturas de tópicos** e recebam publicações originadas por **aplicações locais autorizadas**, com retorno ao destino Telegram correto.

Esta função é **diferente** de:
- aprovação de uma pessoa para acessar um bot Telegram;
- alertas do Scheduler por mudança de estado;
- envio periódico de conteúdo ou mailings (`IN-013`), que permanecem em backlog.

## Obrigações funcionais documentadas

| Tema | Requisito preservado do consolidado | Parte ainda sem especificação |
|---|---|---|
| Assinatura | Usuário autorizado pode assinar e cancelar tópicos | Identidade de tópico, operações, formato de entrada/saída e resultado |
| Publicação | Aplicação local autorizada pode publicar eventos | Interface, identificador do emissor, autenticação, envelopes e erros |
| Autorização | Verificar quem pode publicar/assinar/receber cada tópico | Políticas e escopos por bot, tópico e usuário |
| Sequência e persistência | Preservar sequência por tópico/assinante, histórico e entregas entre reinícios | Chaves, transações, ordenação, schema e retenção |
| Entrega | Aceitar publicação **não comprova leitura pelo destinatário** | Confirmação, retry, falha, deduplicação e reconciliação |
| Isolamento | Resposta/entrega não pode vazar entre bots ou destinatários | Regras técnicas de correlação e autorização |

As operações e campos **não são normativos** enquanto não houver aprovação. Não extrapolar o protocolo MessagePack de mídia nem os estados da outbox Telegram para o protocolo ainda inexistente de publicação de eventos.

## Critérios de validação futuros

- Assinaturas sobrevivem a reinícios; cancelamento impede entregas futuras conforme regra a definir.
- Publicador não autorizado não cria publicação válida; destinatário só recebe temas permitidos.
- Ordenação, histórico e confirmação são verificados por testes de concorrência e reinício, quando houver contrato aprovado.
- Separar testes de assinatura, aprovação de acesso e alertas, porque são fluxos distintos.

**Nenhum teste executável foi encontrado nesta branch documental.**

## Bloqueios

Definir transporte e contrato de publicação; permissão, tópico e assinatura; modelo de persistência; regras de entrega e confirmação; semântica de cancelamento; replay, retenção, ordem e compatibilidade com o isolamento por cliente. O objetivo permanece documentado, **não liberado para implementação contratual**.

## Referências e rastreabilidade

- [Regras consolidadas — seção 9](../regras-consolidadas-sabia.md), fonte de `OBJ-009` e exclusão de `IN-013`
- [Scheduler e alertas — assunto distinto](../fluxos/FLUX-003-monitoramento-agendado.md)
- [Segurança e isolamento](../arquitetura/ARQ-006-seguranca-autorizacao.md)
- [Gate do MVP](../referencia/REF-006-gate-mvp.md)

A origem histórica cita `docs/charter/objectives/OBJ-009-assinatura-e-entrega-de-eventos-telegram.md`, mas esse documento **não está nesta branch**; não foi reconstruído.
