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
- restringir o socket de mídia a processos locais autorizados;
- impedir que conhecer um `request_id` seja suficiente para abrir o canal de mídia;
- proteger credenciais e acesso ao PostgreSQL contra uso por identidades não autorizadas;

## Entradas

Identidade do cliente, Telegram User ID e chat de origem.

## Saídas

Autorizado ou negado.

## Interfaces e contratos

- [CTR-0004 — Auditoria e logs](../contratos/auditoria-logs.md)

## Eventos

Toda tentativa processada deve permitir auditoria do resultado sem registrar segredos.

## Persistência

Whitelists e referências de segredo vêm da configuração. Tokens reais não são versionados. Mídia persistida e as credenciais/roles PostgreSQL devem permanecer acessíveis somente à identidade operacional e identidades locais explicitamente autorizadas pelo modelo de permissões.

## Restrições

O módulo não concede shell arbitrário nem eleva o processo principal a root. O Media Ingest não expõe listener TCP; autorização do socket é adicional à validação de `request_id`.

## Critérios de aceite

- usuário não autorizado não chega ao roteador;
- clientes distintos usam listas distintas;
- tokens não aparecem em logs/auditoria;
- processo opera com identidade Linux dedicada;
- conexão ao socket de mídia é recusada quando o sistema operacional não autoriza o processo local;
- conteúdo BLOB não aparece em logs.

## Implementação relacionada

Prevista para os pacotes internos de security/config.
