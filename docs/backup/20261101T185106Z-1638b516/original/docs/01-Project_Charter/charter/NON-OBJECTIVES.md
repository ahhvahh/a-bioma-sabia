# Não objetivos

| ID | Não objetivo | Justificativa |
|---|---|---|
| NOBJ-001 | Oferecer shell remoto arbitrário ou executar comandos/caminhos fornecidos livremente pelo usuário. | O projeto opera somente sobre operações previamente cadastradas. |
| NOBJ-002 | Expor webhook público para receber updates do Telegram no primeiro escopo operacional. | O canal inicial utiliza conexões de saída por long polling. |
| NOBJ-003 | Implementar processadores complexos de imagem, vídeo ou áudio no primeiro MVP. | O primeiro MVP entrega a infraestrutura para integração, jobs e transporte de mídia; processadores complexos são evolução posterior. |
| NOBJ-004 | Disponibilizar HTTP/REST, gRPC, TCP ou outros canais remotos como interface do primeiro MVP. | Esses canais são possibilidades de evolução e não pertencem à fronteira atual. |
| NOBJ-005 | Conceder a aplicações locais acesso direto ao banco de dados ou aos diretórios internos do Sabiá. | As integrações locais devem ocorrer somente pelas interfaces explicitamente autorizadas. |

## Exclusão revogada

- [NOBJ-006 — Assinaturas criadas por usuários (cancelada)](non-objectives/NOBJ-006-assinaturas-excluidas-cancelado.md): não é mais não objetivo ativo após a decisão de 21/10/2026. A assinatura **por evento** é agora [OBJ-009](objectives/OBJ-009-assinatura-e-entrega-de-eventos-telegram.md).
- Assinaturas e mailings **periódicos** descritos em [IN-013](../scope/in-scope/IN-013-assinaturas-periodicas.md) permanecem em backlog até revisão específica do Scope; não confundir com o novo objetivo de publicações por evento.

Um não objetivo não deve ser removido silenciosamente. Se passar a ser necessário na execução corrente, trate como mudança de escopo.
