# Interfaces e serviços — inventário

> **Estado:** interfaces **planejadas/documentadas**, nenhuma comprovadamente executável na branch `develop`, que não contém `/src`. Caminhos, detalhes de implantação e contratos em `refinement` não devem ser tratados como API estável.

## Canais identificados na documentação

| Interface | Consumidor e finalidade | Transporte/endereço | Acesso | Estado |
|---|---|---|---|---|
| Telegram Bot API | Bots e pessoas; entrada e resposta | HTTPS externo, `getUpdates` por long polling | Token por cliente, whitelist de usuários/chats | Especificado parcialmente; não implementado |
| Socket de mídia integral | Produtores locais; upload de até 20 MB | Unix socket + MessagePack; caminho pendente | `sabia:abioma`, `0660` previstos | CTR-0008 em `refinement` |
| Socket de mídia fracionada | Produtores locais; arquivos até 50 MB | Outro Unix socket + MessagePack; caminho pendente | `sabia:abioma`, `0660` previstos | CTR-0009 em `refinement` |
| Protocolo do processador | Scripts Bash cadastrados; progresso e resultado | `stdin` e `stdout` JSON Lines | Registro local obrigatório | CTR-0005 previsto |
| CLI administrativa | Operador; cadastro do catálogo de ações | CLI local, sintaxe ainda não consolidada | Controle administrativo local a definir | Prevista em Scope; não implementada |
| PostgreSQL (interno) | Persistência do serviço | Conexão de banco ainda a definir | Role e credenciais restritas ao serviço | Não exposto como interface final |

Não existem endpoints REST/HTTP próprios, servidor MCP, gRPC, WebSocket, fila/broker de rede ou listener TCP **implementados ou especificados como interfaces públicas** nesta versão documental. O uso de HTTPS com a **Telegram Bot API externa** não representa uma API HTTP servida pelo Sabiá.

## Telegram — comandos previstos

| Operação | Entrada | Saída esperada | Erros/limitações |
|---|---|---|---|
| `/start`, `/menu` | Identidade do usuário e chat, recebidas no update | Catálogo/menus autorizados | Texto final, caso vazio e erros ainda pendentes |
| `/help`, `/status` | Update de comando | Mensagem associada ao cliente | Contratos operacionais pendentes |
| `/cpu`, `/memory`, `/disk`, `/uptime` | Comando do cliente Bioma | Resultado de operação cadastrada | Argumentos, validações e erros pendentes |
| `/run`, `/jobs`, `/job` | Comando e possíveis argumentos | Execução/consulta conforme cliente | Sem sintaxe e resposta finais |

Nenhum dos comandos acima está disponível para teste nesta branch. Lista de escopo e navegação: [MVP](especificacao/requisitos/mvp.md) e [uso humano](guia-usuario.md).

## Socket de mídia integral — CTR-0008 (proposta)

| Operação | Entrada MessagePack | Sucesso MessagePack | Erros documentados |
|---|---|---|---|
| Upload integral | `version=1`, `request_id` UUID v4, `name`, `content_type` opcional, `data` binário (até 20.000.000 bytes) | `status=accepted`, `request_id`, `media_id` | `unsupported_version`, `unknown_request`, `invalid_message`, `invalid_content_type`, `media_too_large`, `persistence_failed`, `not_authorized` |

Uma conexão prevê um objeto de pedido e um objeto de resposta. O ACK deve ser posterior ao commit. A mensagem não contém caminho de arquivo, cliente Telegram nem destino de entrega. [Contrato completo](especificacao/contratos/ingestao-midia-messagepack.md).

## Socket de mídia fracionada — CTR-0009 (proposta)

| Operação | Entrada MessagePack | Sucesso MessagePack | Erros principais |
|---|---|---|---|
| Abrir mídia | `version=1`, `request_id`, `name`, `content_type` opcional, `total_bytes` (até 50.000.000) | `accepted`, `media_id`, `next_sequence_id`, contadores | `unknown_request`, `media_too_large`, `invalid_content_type` |
| Enviar parte | `media_id`, `sequence_id`, `data` (até 5.000.000 bytes) | `accepted`, sequência seguinte, `received_bytes`, `completed` | `unknown_media`, `chunk_too_large`, `sequence_gap`, `sequence_already_received`, `total_bytes_exceeded`, `persistence_failed` |

A ordem começa em `sequence_id=1`; completude é reconhecida automaticamente ao alcançar `total_bytes`. [Contrato completo](especificacao/contratos/ingestao-midia-fracionada.md).

## Processador interno e administração

| Operação | Entrada/formato | Saída | Limitação |
|---|---|---|---|
| Executar processador registrado | `request_id` UUID v4 em uma linha no `stdin`; argumentos já validados | Eventos `loading` / `finally` em JSON Lines UTF-8 no `stdout` | Somente Bash cadastrado no MVP; timeout documentado de 2h |
| Cadastrar ação via CLI | Contrato de comando não definido | Retorno não definido | Fluxo conceitual, nenhum comando executável para copiar |

Referências: [CTR-0005](especificacao/contratos/processador-assincrono.md), [fluxo administrativo FLW-007](01-Project_Charter/scope/flows/FLW-007-cadastro-administrativo-e-execucao-de-acao-persistente.md) e [detalhamento técnico](referencia-contratos.md).

**Categorias de falha:** erros de protocolo/validação (MessagePack, versão, sequência), autorização (`not_authorized`), domínio/correlação (`unknown_request`, `unknown_media`) e persistência (`persistence_failed`). Tratamento de falhas reais de transporte ainda não é verificável.
