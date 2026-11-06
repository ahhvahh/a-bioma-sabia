<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ARQ-006 — Segurança e autorização

**ID:** `ARQ-006` · **Origem:** `MOD-0006` · **Estado histórico:** `refined`  
**Implementação:** prevista, sem evidência de código na branch

## Objetivo

Impedir que clientes não autorizados ou entradas arbitrárias alcancem operações locais.

> **Precedência documental:** os requisitos que citam o grupo `abioma` abaixo reproduzem a origem histórica `MOD-0006`. A diretriz posterior para os sockets do serviço propõe `absabia`; a divergência ainda não foi aprovada como configuração implantada. Consulte a seção de isolamento complementar.

## Responsabilidades

- validar Telegram User ID na whitelist do cliente;
- validar chat quando `allowed_chats` estiver configurado;
- impedir execução antes da autorização;
- operar como usuário Linux `sabia`;
- impedir registro de tokens, senhas, segredos e credenciais;
- preservar isolamento entre clientes;
- restringir os sockets locais de mídia a processos executados por identidades Linux pertencentes ao grupo `abioma`;
- impedir que conhecer um `request_id` seja suficiente para abrir o canal de mídia;
- proteger credenciais e acesso ao PostgreSQL contra uso por identidades não autorizadas;
- separar o acesso operacional do usuário `sabia` do acesso às interfaces locais concedido ao grupo `abioma`;
- permitir que membros do grupo `abioma` invoquem a aplicação somente pelo link local configurado para essa finalidade;

## Entradas e saídas

**Entrada:** Identidade do cliente, Telegram User ID e chat de origem.

**Saída:** Autorizado ou negado.

## Eventos

Toda tentativa processada deve permitir auditoria do resultado sem registrar segredos.

## Persistência

Whitelists e referências de segredo vêm da configuração. Tokens reais não são versionados. O usuário Linux `sabia` é a identidade operacional da aplicação e mantém o acesso necessário ao PostgreSQL e aos diretórios operacionais do sistema. A associação ao grupo `abioma` autoriza o uso das interfaces locais — sockets e link de invocação da aplicação — mas não concede, por si só, acesso direto ao PostgreSQL nem aos diretórios internos do Sabiá.

## Restrições

O módulo não concede shell arbitrário nem eleva o processo principal a root. O Media Ingest não expõe listener TCP; autorização do socket é adicional à validação de `request_id`. Os sockets pertencem ao usuário `sabia`, ao grupo `abioma` e não concedem acesso a usuários fora desse grupo.

## Critérios de aceite

- usuário não autorizado não chega ao roteador;
- clientes distintos usam listas distintas;
- tokens não aparecem em logs/auditoria;
- processo opera com identidade Linux dedicada;
- conexão aos sockets de mídia é permitida ao usuário `sabia` e a processos cuja identidade efetiva pertença ao grupo `abioma`, e recusada aos demais;
- conteúdo BLOB não aparece em logs;
- pertencer ao grupo `abioma` não concede acesso direto ao banco de dados ou aos diretórios internos da aplicação;
- membros de `abioma` podem acessar a aplicação pelo link local configurado.

## Dependências e contratos

- [DSG-0002 — Componentes do Sabiá Core](ARQ-002-componentes-core.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../decisoes/DEC-008-menor-privilegio-autorizacao.md)
- [ADR-0004 — Registro explícito de scripts](../decisoes/DEC-004-registro-explicito-scripts.md)
- [CTR-0004 — Auditoria e logs](../especificacao/contratos/auditoria-logs.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../especificacao/contratos/ingestao-midia-messagepack.md)

## Implementação prevista

Prevista para os pacotes internos de security/config. **Não verificada no código-fonte desta branch.**

## Pendências de integração

**Conflito documental:** o módulo histórico usa `abioma` para sockets; a [diretriz de segurança](../seguranca-acessos.md) propõe `absabia` para acesso aos UDS da aplicação e `abioma_db` para PostgreSQL. Manter este ponto como pendência antes de implementar permissões. As regras de whitelists YAML também precisam de conciliação com autorizações dinâmicas.

## Diretriz complementar de isolamento e administração (proposta posterior)

Conteúdo absorvido de [Segurança e acessos — documento de origem](../seguranca-acessos.md). A especificação é **planejada**, sem contas, sockets ou privilégios implantados comprovados. Quando há conflito com `MOD-0006`, nenhuma implementação foi presumida.

### Grupos e usuários

| Identidade | Responsabilidade | Acesso |
|---|---|---|
| `sabia` (usuário dedicado) | Executar serviço sem root e sem login interativo | Associado a `absabia` e `abioma_db`; acessa dados e arquivos internos |
| `absabia` (grupo) | Usuários e aplicações locais clientes do Sabiá | Sockets UDS da aplicação, não o banco |
| `abioma_db` (grupo) | Restrição do socket PostgreSQL | Somente `sabia` e administração de banco estritamente necessária |
| Operador local | Executar `sabiactl` | Membro de `absabia`, sem credenciais SQL |
| Usuário Telegram | Ações autorizadas remotamente | Autorização por cliente lógico, usuário e chat |

Um usuário externo ao serviço, mas residente no mesmo servidor, poderá acessar os sockets da aplicação **depois de ser incluído em `absabia`**. Isso não lhe concederá acesso ao banco, à conta `sabia` nem aos diretórios internos.

### Sockets e diretórios — valores pretendidos

| Recurso | Dono/grupo | Permissões planejadas |
|---|---|---|
| Sockets UDS administrativos e de mídia | `sabia:absabia` | `0660`, em diretórios protegidos |
| Socket PostgreSQL | `postgres:abioma_db` | A restringir, por exemplo `0770`; validar no cluster instalado |
| Scripts, configurações secretas e diretórios internos | Identidade de serviço | Sem acesso direto pelo grupo consumidor `absabia` |

Os caminhos e modos definitivos permanecem pendentes. Além da permissão do arquivo socket, os diretórios pais devem permitir travessia apenas aos usuários adequados.

### PostgreSQL: acesso exclusivo por UDS

O serviço deverá usar conexão pelo socket local do PostgreSQL, com banco lógico e role SQL próprios. Configurar `unix_socket_directories`, `unix_socket_group`, `unix_socket_permissions` e regras `local` em `pg_hba.conf`. Preferir autenticação `peer` mapeada ao usuário de serviço, quando compatível, e privilégio mínimo SQL.

**Importante:** as opções de socket são da instância PostgreSQL, não de cada banco lógico. Se a instância for compartilhada, será preciso validar impacto em outros serviços ou usar cluster dedicado para obter isolamento estrito. Restringir também conexões TCP, pois permissões de UDS **não** protegem portas de rede nem substituem autenticação e privilégios SQL.

Documentação: [configuração PostgreSQL](https://www.postgresql.org/docs/current/runtime-config-connection.html), [regras `pg_hba.conf`](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html), [permissões UDS no Linux](https://man7.org/linux/man-pages/man7/unix.7.html).

### Operações administrativas e scripts

O `sabiactl` usará apenas a API UDS do serviço. A aplicação não recebe credenciais SQL nem acesso direto às pastas internas. O serviço deve validar identidade do cliente, autorização por operação e dados enviados.

- **Status, conexões e diagnósticos:** respostas sem segredos nem dados desnecessários.
- **Menus e configurações:** autorização de alteração, validação, auditoria e proteção contra alterações concorrentes.
- **Upload de scripts:** limitar tamanho, nome e conteúdo; impedir traversal (`../`), caminhos arbitrários, links perigosos e sobrescritas de arquivos.
- **Ativação:** receber e validar arquivo não implica executá-lo. Exigir etapa de aprovação/ativação conforme política a ser definida.
- **Autenticação local:** conferir permissão do UDS e considerar identificação Linux do par (como `SO_PEERCRED`), além de regras por operação.

O grupo `absabia` autoriza **conexão**, não necessariamente qualquer alteração administrativa irrestrita. A política detalhada de operações privilegiadas está pendente.

### Autorizar solicitações de acesso Telegram

Um usuário Telegram que envia `/start` torna-se contato conhecido, **não automaticamente autorizado**. O `sabiactl` deverá permitir ao operador selecionar o bot e decidir sobre solicitações em fila. Os botões de aprovação/rejeição são ações administrativas: `→` ou `←` **apenas preparam** uma decisão, e `Y` confirma após mostrar a identidade completa e o bot.

O serviço deverá limitar a aprovação por bot e identidade Telegram estável, impor autorização administrativa além da participação em `absabia`, registrar a decisão e impedir aprovações concorrentes contraditórias. Rejeitar retira da lista de pendências, mas preserva histórico. O banco nunca será acessado diretamente pela aplicação de terminal.

Detalhes do fluxo: `Solicitações de acesso` (referência original ausente: `docs/solicitacoes-acesso.md`; ver [entidade de solicitação proposta](../persistencia/entidades/telegram-access-request.md) para o modelo de dados, não para a tela). O modelo de `allowed_users` no YAML ainda precisa ser reconciliado com aprovações dinâmicas no PostgreSQL.

### Compatibilidade documental

[CFG-0001](../referencia/REF-001-modelo-configuracao.md), [CTR-0008](../contratos/CONTR-008-ingestao-midia-messagepack.md), [CTR-0009](../contratos/CONTR-009-ingestao-midia-fracionada.md) e [MOD-0006](ARQ-006-seguranca-autorizacao.md) ainda fixam o grupo legado `abioma`. A decisão nova usa `absabia`. Reconciliar os contratos normativos antes de implementar.

Veja também [interfaces](../interfaces-servicos.md), [instalação](../guias/GUIA-001-preparar-instalacao.md) e [pendências](../pendencias.md).

## Rastreabilidade

[MOD-0006 — especificação original](../especificacao/modulos/seguranca.md). O identificador original permanece válido para documentos históricos.
