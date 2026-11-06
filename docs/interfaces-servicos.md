<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Interfaces e serviços — inventário

> **Conteúdo lido e migrado para:** [ARQ-011 — Canais](arquitetura/ARQ-011-inventario-canais-servicos.md) e [INTF-TERM-001 — Terminal](interfaces/terminal/INTF-TERM-001-sabiactl.md). Os contratos UDS administrativos seguem pendentes.

> **Estado:** interfaces **planejadas/documentadas**, nenhuma comprovadamente executável na branch `develop`, que não contém `/src`. Caminhos, detalhes de implantação e contratos em `refinement` não devem ser tratados como API estável.

## Canais identificados na documentação

| Interface | Consumidor e finalidade | Transporte/endereço | Acesso | Estado |
|---|---|---|---|---|
| API UDS administrativa | `sabiactl` e clientes locais; status, testes, menus, configurações e scripts | Socket Unix; caminho/protocolo pendentes | Grupo `absabia` e política por operação | Novo requisito; não implementado |
| Telegram Bot API | Bots e pessoas; entrada e resposta | HTTPS externo, `getUpdates` por long polling | Token por cliente, whitelist de usuários/chats | Especificado parcialmente; não implementado |
| Socket de mídia integral | Produtores locais; upload de até 20 MB | Unix socket + MessagePack; caminho pendente | `sabia:absabia`, `0660` solicitados; contrato antigo usa `abioma` | CTR-0008 em `refinement` |
| Socket de mídia fracionada | Produtores locais; arquivos até 50 MB | Outro Unix socket + MessagePack; caminho pendente | `sabia:absabia`, `0660` solicitados; contrato antigo usa `abioma` | CTR-0009 em `refinement` |
| Protocolo do processador | Scripts Bash cadastrados; progresso e resultado | `stdin` e `stdout` JSON Lines | Registro local obrigatório | CTR-0005 previsto |
| `sabiactl` | Operador autorizado; terminal navegável | Cliente da API UDS | Grupo `absabia` | Novo requisito; não implementado |
| PostgreSQL (interno) | Exclusivo do serviço `sabia` | UDS PostgreSQL; caminho pendente | Grupo `abioma_db` + autenticação SQL | Novo requisito; não exposto aos usuários |

Não existem endpoints REST/HTTP próprios, servidor MCP, gRPC, WebSocket, fila/broker de rede ou listener TCP **implementados ou especificados como interfaces públicas** nesta versão documental. O uso de HTTPS com a **Telegram Bot API externa** não representa uma API HTTP servida pelo Sabiá. A nova API UDS administrativa não é HTTP nem TCP por definição.

## API UDS administrativa — novo requisito

Esta API será consumida pelo `sabiactl` e por outras aplicações locais autorizadas pelo grupo `absabia`. **Não está implementada**, e nomes definitivos, envelope, framing, codificação, erros e caminhos ainda precisam de contrato.

| Capacidade | Entrada prevista | Saída prevista | Falhas a tratar |
|---|---|---|---|
| Consultar saúde/estado | Identificador opcional do componente | Status, uptime e diagnósticos | Serviço indisponível, acesso negado |
| Conexões e usuários | Filtros de tipo/estado | UDS ativos, sessões locais, contatos Telegram e clientes configurados, contados separadamente | Falha de medição, filtro inválido |
| Serviços e processadores | Filtros e identificação | Configurações e estados conhecidos | Não encontrado, não autorizado |
| Executar testes | Alvo e verificação permitida | Resultado, duração e erros | Timeout, permissão |
| Gerenciar menus e ações | Ação, categoria e campos validados | Registro persistido/consultado | Conflito, dados inválidos |
| Configurações | Chaves/valores permitidos | Resultado sem revelar segredos | Sem autorização, valor inválido |
| Enviar script | Arquivo e metadados | Identificador de upload e validação | Nome, tamanho, conteúdo, armazenamento |
| Incorporar/ativar script | Identificador já validado | Estado da incorporação | Não validado, não autorizado |

A camada UDS é exclusivamente a **porta de entrada**. Somente o serviço `sabia` consulta o PostgreSQL (via UDS protegido por `abioma_db`) e grava diretórios internos. A inclusão em `absabia` permite estabelecer conexão; alterações sensíveis e ativação de scripts exigem política de autorização adicional.

**Observação:** `MessagePack` é o protocolo já especificado para *mídia*, mas não foi definido como contrato da nova API administrativa. Não extrapolar sua codificação até aprovação formal.

**Divergência:** os contratos antigos CTR-0008/0009 usam o grupo `abioma`. O novo requisito usa `absabia` e ainda exige atualização dos contratos originais.

## Solicitações de acesso por bot — requisito de administração

A UI do `sabiactl` usará uma fila por bot, com consulta paginada de pedidos pendentes e ações de aprovação ou rejeição. A alteração exige confirmação humana (`Y` no terminal), autorização administrativa por cliente e gravação transacional pelo serviço. A API UDS deve suportar listar bots/pendências, listar solicitações de um bot, aprovar e rejeitar pedidos, tratar conflitos entre operadores e retornar estados auditáveis.

**Esses são comportamentos propostos, não nomes de métodos, formato de transporte ou códigos aprovados.** O `sabiactl` não acessa PostgreSQL diretamente. O detalhe da experiência e dos dados necessários está em `solicitacoes-acesso.md` (documento visual histórico ausente na branch; a [entidade proposta de solicitação](persistencia/entidades/ENT-013-telegram-access-request.md) não substitui o fluxo de interface).

## Telegram — comandos previstos

| Operação | Entrada | Saída esperada | Erros/limitações |
|---|---|---|---|
| `/start`, `/menu` | Identidade do usuário e chat, recebidas no update | Catálogo/menus autorizados | Texto final, caso vazio e erros ainda pendentes |
| `/help`, `/status` | Update de comando | Mensagem associada ao cliente | Contratos operacionais pendentes |
| `/cpu`, `/memory`, `/disk`, `/uptime` | Comando do cliente Bioma | Resultado de operação cadastrada | Argumentos, validações e erros pendentes |
| `/run`, `/jobs`, `/job` | Comando e possíveis argumentos | Execução/consulta conforme cliente | Sem sintaxe e resposta finais |

Nenhum dos comandos acima está disponível para teste nesta branch. Lista de escopo e navegação: [MVP](referencia/REF-002-escopo-primeiro-mvp.md) e [uso humano](guia-usuario.md).

## Socket de mídia integral — CTR-0008 (proposta)

| Operação | Entrada MessagePack | Sucesso MessagePack | Erros documentados |
|---|---|---|---|
| Upload integral | `version=1`, `request_id` UUID v4, `name`, `content_type` opcional, `data` binário (até 20.000.000 bytes) | `status=accepted`, `request_id`, `media_id` | `unsupported_version`, `unknown_request`, `invalid_message`, `invalid_content_type`, `media_too_large`, `persistence_failed`, `not_authorized` |

Uma conexão prevê um objeto de pedido e um objeto de resposta. O ACK deve ser posterior ao commit. A mensagem não contém caminho de arquivo, cliente Telegram nem destino de entrega. [Contrato completo](contratos/CONTR-008-ingestao-midia-messagepack.md).

## Socket de mídia fracionada — CTR-0009 (proposta)

| Operação | Entrada MessagePack | Sucesso MessagePack | Erros principais |
|---|---|---|---|
| Abrir mídia | `version=1`, `request_id`, `name`, `content_type` opcional, `total_bytes` (até 50.000.000) | `accepted`, `media_id`, `next_sequence_id`, contadores | `unknown_request`, `media_too_large`, `invalid_content_type` |
| Enviar parte | `media_id`, `sequence_id`, `data` (até 5.000.000 bytes) | `accepted`, sequência seguinte, `received_bytes`, `completed` | `unknown_media`, `chunk_too_large`, `sequence_gap`, `sequence_already_received`, `total_bytes_exceeded`, `persistence_failed` |

A ordem começa em `sequence_id=1`; completude é reconhecida automaticamente ao alcançar `total_bytes`. [Contrato completo](contratos/CONTR-009-ingestao-midia-fracionada.md).

## Processador interno e administração

| Operação | Entrada/formato | Saída | Limitação |
|---|---|---|---|
| Executar processador registrado | `request_id` UUID v4 em uma linha no `stdin`; argumentos já validados | Eventos `loading` / `finally` em JSON Lines UTF-8 no `stdout` | Somente Bash cadastrado no MVP; timeout documentado de 2h |
| Cadastrar ação via CLI | Contrato de comando não definido | Retorno não definido | Fluxo conceitual, nenhum comando executável para copiar |

Referências: [CTR-0005](contratos/CONTR-005-processador-assincrono.md), `FLW-007` (documento histórico ausente nesta branch; ver [terminal proposto](interfaces/terminal/INTF-TERM-001-sabiactl.md)) e [detalhamento técnico](referencia-contratos.md).

**Categorias de falha:** erros de protocolo/validação (MessagePack, versão, sequência), autorização (`not_authorized`), domínio/correlação (`unknown_request`, `unknown_media`) e persistência (`persistence_failed`). Tratamento de falhas reais de transporte ainda não é verificável.
