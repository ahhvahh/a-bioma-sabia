# Fora do escopo

| ID | Exclusão | Justificativa | Relacionado a |
|---|---|---|---|
| OUT-001 | Execução de shell, comando ou caminho arbitrário recebido do Telegram. | Operações devem ser previamente cadastradas e autorizadas. | - |
| OUT-002 | Recebimento de updates do Telegram por webhook público no primeiro MVP. | O canal inicial utiliza long polling. | - |
| OUT-003 | Exposição de HTTP/REST, gRPC ou TCP como canal remoto do primeiro MVP. | São possibilidades de evolução posterior e não fazem parte da fronteira atual. | - |
| OUT-004 | Implementação, no primeiro MVP, de processadores complexos de imagem, vídeo e áudio. | O MVP fornece a infraestrutura de jobs e transporte, mas não esses processadores especializados. | - |
| OUT-005 | Acesso direto de aplicações locais ao PostgreSQL, credenciais ou diretórios internos do Sabiá como mecanismo de integração. | Integrações locais devem ocorrer somente pelas interfaces autorizadas. | - |
| OUT-006 | Execução do processo principal como root. | O projeto adota menor privilégio e identidade Linux dedicada. | - |
| OUT-007 | Compartilhamento implícito de comandos, scripts, permissões ou credenciais entre clientes Telegram. | Cada cliente possui isolamento lógico próprio. | - |
| OUT-008 | Cadastro ou alteração remota de caminho de script, executável ou processador por usuário Telegram. | Administração dessas definições ocorre somente pela CLI local do Sabiá. | - |
| OUT-009 | Execução de assinaturas criadas pelo usuário e mailing periódico no primeiro MVP. | A capacidade foi aceita como IN-013 com `Disposition: backlog` e não integra a execução corrente. | IN-013 |

Itens OUT não devem ser removidos silenciosamente. Quando uma solicitação os contradizer, trate como possível mudança de escopo.
