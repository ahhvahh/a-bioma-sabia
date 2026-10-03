# Menor privilégio e autorização explícita

![ADR](https://img.shields.io/badge/ADR-ADR--0008-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

O Sabiá oferece acesso remoto a operações locais do servidor por Telegram.

## Problema

Limitar quem pode acionar operações e quais privilégios o processo possui no Linux.

## Restrições

- processo não deve executar como root;
- usuário Linux dedicado da aplicação: `sabia`;
- grupo Linux autorizado para uso das interfaces locais: `abioma`;
- cada cliente possui whitelist própria de Telegram User IDs;
- chats também podem ser restringidos;
- tokens, senhas e segredos não podem aparecer em logs;
- tokens não podem ser versionados no Git.

## Opções consideradas

### Processo privilegiado e autorização implícita

Incompatível com os requisitos de segurança.

### Menor privilégio e listas explícitas

Restringe identidade local e acesso remoto.

## Decisão

Executar o serviço como usuário Linux `sabia` e autorizar comandos remotos somente após validar as listas do cliente correspondente. O usuário `sabia` mantém o acesso operacional necessário ao PostgreSQL e aos diretórios da aplicação. Interfaces locais destinadas a outras aplicações — sockets e o link local configurado para invocação — são acessíveis a identidades pertencentes ao grupo Linux `abioma`. Pertencer a `abioma` não concede acesso direto ao PostgreSQL nem aos diretórios internos. Tokens vêm de variável de ambiente ou arquivo protegido, nunca de configuração versionada com valor real.

## Justificativa

Limita impacto de comprometimento e aplica isolamento por cliente.

## Consequências

- scripts que precisem privilégios extras exigirão mecanismo explícito fora deste ADR;
- autorização ocorre antes do Command Router executar a operação;
- auditoria registra identidade e resultado sem segredos;
- sockets locais compartilhados com aplicações usam owner `sabia`, group `abioma` e mode `0660`;
- usuários fora do grupo `abioma` não acessam essas interfaces locais.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](../telegram/multiplos-clientes-isolados.md)
- [ADR-0004 — Registro explícito de scripts](../execucao/registro-explicito-de-scripts.md)

## Critérios de validação

- usuário Telegram fora da whitelist não executa comando;
- o processo principal não roda como root;
- tokens reais não aparecem em YAML versionado nem em logs;
- membros do grupo `abioma` conseguem usar as interfaces locais autorizadas sem receber acesso direto às credenciais, ao PostgreSQL ou aos diretórios internos da aplicação.
