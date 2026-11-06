<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ARQ-011 — Inventário de canais e serviços do Sabiá

**Situação:** arquitetura proposta. Na branch documental não foi comprovado servidor, CLI ou transporte em execução. Os nomes e permissões são especificações a validar, não endpoints implantados.

## Canais identificados

| Canal | Consumidor | Transporte e responsabilidade | Acesso previsto | Documento normativo relacionado |
|---|---|---|---|---|
| API administrativa | `sabiactl` e aplicações locais | UDS do Sabiá para consulta e administração; caminho, framing e serialização não definidos | Grupo `absabia` para conexão; autorização própria para alterações | [Interface do terminal](../interfaces/terminal/INTF-TERM-001-sabiactl.md) |
| Telegram Bot API | Bots `bioma`, `tools`, `alerts` | HTTPS **externo** por `getUpdates` e operações de envio | Token e autorização isolados por bot/usuário/chat | [Adaptador Telegram](ARQ-008-adaptador-telegram.md), [fluxo](../fluxos/FLUX-001-comando-telegram.md) |
| Ingestão integral de mídia | Produtores locais autorizados | UDS com MessagePack, uma solicitação/uma resposta; até **20.000.000 bytes** | Proposta `absabia`, contrato antigo `abioma`; permissões dependem de decisão | [CONTR-008](../contratos/CONTR-008-ingestao-midia-messagepack.md) |
| Ingestão fracionada | Produtores locais autorizados | Outro UDS MessagePack, total até **50.000.000 bytes**, partes até **5.000.000 bytes** | Conflito de grupos ainda não resolvido | [CONTR-009](../contratos/CONTR-009-ingestao-midia-fracionada.md) |
| Processador registrado | Script Bash previamente cadastrado | `stdin` e eventos JSON Lines em `stdout`; timeout absoluto de 2 horas documentado | Somente registro e execução autorizados | [CONTR-005](../contratos/CONTR-005-processador-assincrono.md) |
| PostgreSQL interno | Exclusivamente serviço `sabia` | UDS PostgreSQL, diretório/cluster pendentes | Grupo `abioma_db` e autorização SQL, não acessível diretamente por `sabiactl` | [Configuração](../persistencia/configuracao-banco.md) |

**Não há interface pública própria REST/HTTP, gRPC, WebSocket, MCP ou listener TCP comprovada/especificada neste lote.** A utilização de HTTPS com Telegram é o consumo de serviço externo, não uma API HTTP oferecida pelo Sabiá.

## Capacidades previstas da API administrativa

| Capacidade | Entrada esperada em alto nível | Saída esperada em alto nível | Situação |
|---|---|---|---|
| Estado e saúde | Componente opcional | Status, uptime, diagnósticos | Sem operação/payload aprovados |
| Sessões, contatos e clientes | Filtros por tipo/estado | Contagens e listagem separada por origem | Sem operação/payload aprovados |
| Serviços e processadores | Identificação e filtro | Configuração/estado conhecido | Sem operação/payload aprovados |
| Testes | Alvo e verificação permitida | Resultado, duração e falha | Sem contrato de teste UDS |
| Menus e ações | Operação e campos validados | Catálogo/estado persistente | Sem contrato UDS |
| Configurações | Chaves/valores autorizados | Resultado sem segredos | Sem contrato UDS |
| Upload de scripts | Arquivo e metadados | Identificador e relatório de validação | Sem limite/estrutura final |
| Incorporação e ativação | Identificador validado | Resultado de incorporação/ativação | Requer autorização e confirmação humanas |
| Solicitações de acesso por bot | Bot, consulta/decisão de pedido | Lista paginada ou confirmação auditável | Fluxo administrativo ainda sem contrato |

O grupo `absabia` concede **possibilidade de conexão**, não permissão automática para executar mutações. A camada UDS chama o serviço; **não abre acesso direto ao banco**. O MessagePack dos sockets de mídia **não é contrato** da API administrativa.

## Requisitos administrativos específicos

O [CONTR-011](../contratos/CONTR-011-api-administrativa-uds.md) detalha capacidades previstas e lacunas do UDS administrativo. Não extrapolar o MessagePack de mídia para a API do operador; não há framing nem nomes de operações aprovados.

## Pendências e segurança

- Caminhos definitivos, framing, serialização, operações, erros, limites e autenticação da API administrativa são **não verificados**.
- A solicitação de acesso é uma fila própria do Sabiá, não `chat_join_request` do Telegram nem assinatura paga. A fonte `docs/solicitacoes-acesso.md` citada no inventário original **não está presente na branch**; a [entidade proposta](../persistencia/entidades/ENT-013-telegram-access-request.md) não substitui a experiência visual.
- Os dois contratos de mídia ainda registram o grupo Linux histórico `abioma`, em conflito com `absabia`.
- Os comandos `/start`, `/menu`, `/help`, `/status`, `/cpu`, `/memory`, `/disk`, `/uptime`, `/run`, `/jobs` e `/job` são especificados ou propostos conforme o [escopo](../referencia/REF-002-escopo-primeiro-mvp.md); sua disponibilidade executável não foi verificada.

## Rastreabilidade

Fonte original preservada: [Interfaces e serviços — inventário](../interfaces-servicos.md). O fluxo administrativo histórico `FLW-007`, citado na origem, **não está nesta branch**; não foi recriado.
