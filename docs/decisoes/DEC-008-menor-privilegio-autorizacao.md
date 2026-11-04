<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# DEC-008 — Menor privilégio e autorização explícita

**ID:** `DEC-008`  
**ID histórico:** `ADR-0008`  
**Estado documental de origem:** `refined`  
**Versão histórica registrada:** `1`  
**Implementação:** não verificada nesta revisão

> **Conflito em revisão:** nomes de grupos Linux na decisão histórica não correspondem à diretriz recente. Consulte a seção Limitações da revisão antes de aplicar os valores.

## Decisão

Executar o serviço como usuário Linux `sabia` e autorizar comandos remotos somente após validar as listas do cliente correspondente. O usuário `sabia` mantém o acesso operacional necessário ao PostgreSQL e aos diretórios da aplicação. Interfaces locais destinadas a outras aplicações — sockets e o link local configurado para invocação — são acessíveis a identidades pertencentes ao grupo Linux `abioma`. Pertencer a `abioma` não concede acesso direto ao PostgreSQL nem aos diretórios internos. Tokens vêm de variável de ambiente ou arquivo protegido, nunca de configuração versionada com valor real.

## Contexto e problema

O Sabiá oferece acesso remoto a operações locais do servidor por Telegram.

Limitar quem pode acionar operações e quais privilégios o processo possui no Linux.

## Restrições

- processo não deve executar como root;
- usuário Linux dedicado da aplicação: `sabia`;
- grupo Linux autorizado para uso das interfaces locais: `abioma`;
- cada cliente possui whitelist própria de Telegram User IDs;
- chats também podem ser restringidos;
- tokens, senhas e segredos não podem aparecer em logs;
- tokens não podem ser versionados no Git.

## Alternativas consideradas

### Processo privilegiado e autorização implícita

Incompatível com os requisitos de segurança.

### Menor privilégio e listas explícitas

Restringe identidade local e acesso remoto.

## Justificativa

Limita impacto de comprometimento e aplica isolamento por cliente.

## Consequências

- scripts que precisem privilégios extras exigirão mecanismo explícito fora deste ADR;
- autorização ocorre antes do Command Router executar a operação;
- auditoria registra identidade e resultado sem segredos;
- sockets locais compartilhados com aplicações usam owner `sabia`, group `abioma` e mode `0660`;
- usuários fora do grupo `abioma` não acessam essas interfaces locais.

## Critérios de validação

- usuário Telegram fora da whitelist não executa comando;
- o processo principal não roda como root;
- tokens reais não aparecem em YAML versionado nem em logs;
- membros do grupo `abioma` conseguem usar as interfaces locais autorizadas sem receber acesso direto às credenciais, ao PostgreSQL ou aos diretórios internos da aplicação.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](DEC-002-clientes-telegram-isolados.md)
- [ADR-0004 — Registro explícito de scripts](DEC-004-registro-explicito-scripts.md)

## Limitações da revisão

**Conflito identificado:** o ADR histórico exige grupo Linux `abioma` para os sockets, enquanto a [diretriz de segurança mais recente](../seguranca-acessos.md) propõe `absabia` para clientes do Sabiá e `abioma_db` para acesso SQL. O valor `abioma` preservado nesta transcrição histórica não deve ser aplicado como regra operacional sem reconciliar os contratos existentes.

A revisão documental não demonstra implementação, execução de testes ou aprovação de novas regras.

## Fonte e rastreabilidade

Documento histórico preservado: [`ADR-0008` original](../adr/seguranca/menor-privilegio-e-autorizacao.md). O identificador e a versão original foram mantidos para rastrear a decisão, sem renumerar outros ADRs.
