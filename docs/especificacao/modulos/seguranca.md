# Segurança e autorização

![MOD](https://img.shields.io/badge/MOD-MOD--0006-1f883d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Impedir que clientes não autorizados ou entradas arbitrárias alcancem operações locais.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../../adr/seguranca/menor-privilegio-e-autorizacao.md)
- [ADR-0004 — Registro explícito de scripts](../../adr/execucao/registro-explicito-de-scripts.md)
- [CTR-0004 — Auditoria e logs](../contratos/auditoria-logs.md)
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](../contratos/ingestao-midia-messagepack.md)

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

## Entradas

Identidade do cliente, Telegram User ID e chat de origem.

## Saídas

Autorizado ou negado.

## Interfaces e contratos

- [CTR-0004 — Auditoria e logs](../contratos/auditoria-logs.md)

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

## Implementação relacionada

Prevista para os pacotes internos de security/config.
