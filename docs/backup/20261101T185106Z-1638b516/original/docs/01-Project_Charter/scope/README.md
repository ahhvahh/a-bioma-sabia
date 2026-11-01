# Sabiá — Scope

![Document](https://img.shields.io/badge/ID-SCP--0001-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

## Referência

Charter: [../charter/README.md](../charter/README.md)

## Visão geral

O Sabiá é um serviço Linux do ecossistema Bioma que intermedeia, de forma controlada, usuários no Telegram, operações locais previamente cadastradas, jobs assíncronos, verificações agendadas, alertas e transporte de mídia.

O Telegram é o canal remoto inicial. O Sabiá registra contatos conhecidos a partir da interação com seus clientes, mantém no PostgreSQL o catálogo de ações apresentado aos usuários e oferece uma CLI administrativa local para cadastrar e administrar essas ações.

Aplicações locais podem interagir somente por interfaces explicitamente autorizadas. O Sabiá mantém a correlação entre entrada, processamento, estado persistente e destino de resposta sem transformar mensagens recebidas em shell arbitrário. O registro de um contato não concede, por si só, autorização para executar ações restritas.

## Fronteira

Ver [SYSTEM-BOUNDARY.md](SYSTEM-BOUNDARY.md).

## Atores

Ver [ACTORS.md](ACTORS.md).

## Canais

Ver [CHANNELS.md](CHANNELS.md).

## Contexto do sistema

Ver [SYSTEM-CONTEXT.md](SYSTEM-CONTEXT.md).

## Dentro do escopo

- [IN-001 — Clientes Telegram isolados](in-scope/IN-001-clientes-telegram-isolados.md)
- [IN-002 — Autorização explícita](in-scope/IN-002-autorizacao-explicita.md)
- [IN-003 — Roteamento de operações cadastradas](in-scope/IN-003-roteamento-de-operacoes-cadastradas.md)
- [IN-004 — Jobs assíncronos](in-scope/IN-004-jobs-assincronos.md)
- [IN-005 — Scheduler e alertas](in-scope/IN-005-scheduler-e-alertas.md)
- [IN-006 — Persistência operacional e recovery](in-scope/IN-006-persistencia-operacional-e-recovery.md)
- [IN-007 — Ingestão local de mídia](in-scope/IN-007-ingestao-local-de-midia.md)
- [IN-008 — Entrega de mídia](in-scope/IN-008-entrega-de-midia.md)
- [IN-009 — Operação segura como serviço Linux](in-scope/IN-009-operacao-segura-como-servico-linux.md)
- [IN-010 — Registro persistente de contatos](in-scope/IN-010-registro-persistente-de-contatos.md)
- [IN-011 — Catálogo persistente de ações e menu](in-scope/IN-011-catalogo-persistente-de-acoes-e-menu.md)
- [IN-012 — Administração de ações por CLI](in-scope/IN-012-administracao-de-acoes-por-cli.md)
- [IN-013 — Assinaturas periódicas](in-scope/IN-013-assinaturas-periodicas.md) — `Disposition: backlog`

## Fora do escopo

Ver [out-of-scope/OUT-OF-SCOPE.md](out-of-scope/OUT-OF-SCOPE.md).

## Integrações

- [INT-001 — Telegram Bot API](integrations/INT-001-telegram-bot-api.md)
- [INT-002 — Operações locais cadastradas](integrations/INT-002-operacoes-locais-cadastradas.md)
- [INT-003 — Sabiá](integrations/INT-003-sabia.md)
- [INT-004 — PostgreSQL](integrations/INT-004-postgresql.md)
- [INT-005 — Sabiá](integrations/INT-005-sabia.md)

## Fluxos

- [FLW-001 — Comando Telegram](flows/FLW-001-comando-telegram.md)
- [FLW-002 — Job assíncrono](flows/FLW-002-job-assincrono.md)
- [FLW-003 — Monitoramento agendado e alerta](flows/FLW-003-monitoramento-agendado-e-alerta.md)
- [FLW-004 — Ingestão e entrega de mídia local](flows/FLW-004-ingestao-e-entrega-de-midia-local.md)
- [FLW-005 — Inicialização, recovery e encerramento](flows/FLW-005-inicializacao-recovery-e-encerramento.md)
- [FLW-006 — /start, registro de contato e apresentação do menu](flows/FLW-006-start-registro-de-contato-e-apresentacao-do-menu.md)
- [FLW-007 — Cadastro administrativo e execução de ação persistente](flows/FLW-007-cadastro-administrativo-e-execucao-de-acao-persistente.md)

## Discoveries relacionados

Nenhum Discovery de Project Charter está aberto nesta baseline. Lacunas técnicas de especificação não alteram, neste momento, a fronteira de alto nível.
