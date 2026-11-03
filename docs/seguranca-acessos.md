# Segurança e acessos — isolamento do Sabiá

> **Diretriz de arquitetura, não configuração executada.** O projeto não possui código, contas provisionadas ou sockets ativos. Especificações antigas de mídia ainda citam o grupo `abioma` e precisam ser atualizadas para a nova decisão `absabia`.

## Grupos e usuários

| Identidade | Responsabilidade | Acesso |
|---|---|---|
| `sabia` (usuário dedicado) | Executar serviço sem root e sem login interativo | Associado a `absabia` e `abioma_db`; acessa dados e arquivos internos |
| `absabia` (grupo) | Usuários e aplicações locais clientes do Sabiá | Sockets UDS da aplicação, não o banco |
| `abioma_db` (grupo) | Restrição do socket PostgreSQL | Somente `sabia` e administração de banco estritamente necessária |
| Operador local | Executar `sabiactl` | Membro de `absabia`, sem credenciais SQL |
| Usuário Telegram | Ações autorizadas remotamente | Autorização por cliente lógico, usuário e chat |

Um usuário externo ao serviço, mas residente no mesmo servidor, poderá acessar os sockets da aplicação **depois de ser incluído em `absabia`**. Isso não lhe concederá acesso ao banco, à conta `sabia` nem aos diretórios internos.

## Sockets e diretórios — valores pretendidos

| Recurso | Dono/grupo | Permissões planejadas |
|---|---|---|
| Sockets UDS administrativos e de mídia | `sabia:absabia` | `0660`, em diretórios protegidos |
| Socket PostgreSQL | `postgres:abioma_db` | A restringir, por exemplo `0770`; validar no cluster instalado |
| Scripts, configurações secretas e diretórios internos | Identidade de serviço | Sem acesso direto pelo grupo consumidor `absabia` |

Os caminhos e modos definitivos permanecem pendentes. Além da permissão do arquivo socket, os diretórios pais devem permitir travessia apenas aos usuários adequados.

## PostgreSQL: acesso exclusivo por UDS

O serviço deverá usar conexão pelo socket local do PostgreSQL, com banco lógico e role SQL próprios. Configurar `unix_socket_directories`, `unix_socket_group`, `unix_socket_permissions` e regras `local` em `pg_hba.conf`. Preferir autenticação `peer` mapeada ao usuário de serviço, quando compatível, e privilégio mínimo SQL.

**Importante:** as opções de socket são da instância PostgreSQL, não de cada banco lógico. Se a instância for compartilhada, será preciso validar impacto em outros serviços ou usar cluster dedicado para obter isolamento estrito. Restringir também conexões TCP, pois permissões de UDS **não** protegem portas de rede nem substituem autenticação e privilégios SQL.

Documentação: [configuração PostgreSQL](https://www.postgresql.org/docs/current/runtime-config-connection.html), [regras `pg_hba.conf`](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html), [permissões UDS no Linux](https://man7.org/linux/man-pages/man7/unix.7.html).

## Operações administrativas e scripts

O `sabiactl` usará apenas a API UDS do serviço. A aplicação não recebe credenciais SQL nem acesso direto às pastas internas. O serviço deve validar identidade do cliente, autorização por operação e dados enviados.

- **Status, conexões e diagnósticos:** respostas sem segredos nem dados desnecessários.
- **Menus e configurações:** autorização de alteração, validação, auditoria e proteção contra alterações concorrentes.
- **Upload de scripts:** limitar tamanho, nome e conteúdo; impedir traversal (`../`), caminhos arbitrários, links perigosos e sobrescritas de arquivos.
- **Ativação:** receber e validar arquivo não implica executá-lo. Exigir etapa de aprovação/ativação conforme política a ser definida.
- **Autenticação local:** conferir permissão do UDS e considerar identificação Linux do par (como `SO_PEERCRED`), além de regras por operação.

O grupo `absabia` autoriza **conexão**, não necessariamente qualquer alteração administrativa irrestrita. A política detalhada de operações privilegiadas está pendente.

## Compatibilidade documental

[CFG-0001](especificacao/configuracao/modelo-configuracao.md), [CTR-0008](especificacao/contratos/ingestao-midia-messagepack.md), [CTR-0009](especificacao/contratos/ingestao-midia-fracionada.md) e [MOD-0006](especificacao/modulos/seguranca.md) ainda fixam o grupo legado `abioma`. A decisão nova usa `absabia`. Reconciliar os contratos normativos antes de implementar.

Veja também [interfaces](interfaces-servicos.md), [instalação](instalacao.md) e [pendências](pendencias.md).
