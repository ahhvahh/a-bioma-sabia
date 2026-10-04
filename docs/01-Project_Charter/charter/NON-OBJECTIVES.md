# Não objetivos

| ID | Não objetivo | Justificativa |
|---|---|---|
| NOBJ-001 | Oferecer shell remoto arbitrário ou executar comandos/caminhos fornecidos livremente pelo usuário. | O projeto opera somente sobre operações previamente cadastradas. |
| NOBJ-002 | Expor webhook público para receber updates do Telegram no primeiro escopo operacional. | O canal inicial utiliza conexões de saída por long polling. |
| NOBJ-003 | Implementar processadores complexos de imagem, vídeo ou áudio no primeiro MVP. | O primeiro MVP entrega a infraestrutura para integração, jobs e transporte de mídia; processadores complexos são evolução posterior. |
| NOBJ-004 | Disponibilizar HTTP/REST, gRPC, TCP ou outros canais remotos como interface do primeiro MVP. | Esses canais são possibilidades de evolução e não pertencem à fronteira atual. |
| NOBJ-005 | Conceder a aplicações locais acesso direto ao banco de dados ou aos diretórios internos do Sabiá. | As integrações locais devem ocorrer somente pelas interfaces explicitamente autorizadas. |
| NOBJ-006 | Entregar, no primeiro MVP, o mecanismo completo de assinaturas criadas pelo usuário e mailings periódicos, como `notícias/naval`. | A estrutura é mantida como backlog para evolução posterior sobre contatos e ações persistentes. |

NOBJ-006 é representado operacionalmente pelo item [IN-013](../scope/in-scope/IN-013-assinaturas-periodicas.md), com `Disposition: backlog`.

Um não objetivo não deve ser removido silenciosamente. Se passar a ser necessário na execução corrente, trate como mudança de escopo.
