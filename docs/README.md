# Sabiá — documentação técnica

Esta é a raiz documental do Sabiá, organizada conforme o Architecture Documentation Pipeline (ADP) 1.0.

O Sabiá é um serviço Linux do ecossistema Bioma para integrar aplicações, scripts e serviços locais com clientes Telegram independentes, com foco em segurança, baixo consumo, modularidade e operação sem execução arbitrária de comandos.

Pendências consolidadas: [pendências documentais](pendencias.md).

## Estado do pipeline

A arquitetura principal possui decisões explícitas e desenhos finalizados. O desenvolvimento completo do MVP permanece `BLOCKED` enquanto existirem especificações necessárias em `refinement`, especificações técnicas ausentes ou documentos declarados `refined` cujo conteúdo ainda não seja implementável sem suposição.

A relação normativa e atualizada dos bloqueios permanece centralizada em [docs/pendencias.md](pendencias.md).

### Decisões

- [ADR-0001 — Go, binário único e serviço Linux](adr/runtime/go-binario-unico.md) — `refined`
- [ADR-0002 — Múltiplos clientes Telegram isolados](adr/telegram/multiplos-clientes-isolados.md) — `refined`
- [ADR-0003 — Core independente do Telegram](adr/arquitetura/core-independente-do-telegram.md) — `refined`
- [ADR-0004 — Registro explícito de scripts](adr/execucao/registro-explicito-de-scripts.md) — `refined`
- [ADR-0005 — Telegram Long Polling](adr/telegram/long-polling.md) — `refined`
- [ADR-0006 — Jobs assíncronos](adr/processamento/jobs-assincronos.md) — `refined`
- [ADR-0007 — Scheduler e alertas orientados a estado](adr/monitoramento/scheduler-alertas-estado.md) — `refined`
- [ADR-0008 — Menor privilégio e autorização explícita](adr/seguranca/menor-privilegio-e-autorizacao.md) — `refined`
- [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md) — `refined`
- [ADR-0010 — Política de encerramento de jobs](adr/runtime/encerramento-de-jobs.md) — `refined`
- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](adr/processamento/processadores-assincronos-registrados.md) — `refined`
- [ADR-0012 — Ingestão persistente de mídia por Unix socket](adr/processamento/ingestao-midia-socket-messagepack.md) — `refined`
- [ADR-0013 — Dois canais de ingestão de mídia e upload fracionado](adr/processamento/ingestao-midia-fracionada.md) — `refined`

### Desenhos

- [DSG-0001 — Contexto do Sabiá](desenho/contexto-sabia.md) — `finalized`
- [DSG-0002 — Componentes do Sabiá Core](desenho/componentes-core.md) — `finalized`

### Módulos

- [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md) — `refinement`
- [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md) — `refinement`
- [MOD-0003 — Registro e execução de scripts](especificacao/modulos/scripts.md) — `refined`
- [MOD-0004 — Jobs](especificacao/modulos/jobs.md) — `refined`
- [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md) — `refinement`
- [MOD-0006 — Segurança e autorização](especificacao/modulos/seguranca.md) — `refined`
- [MOD-0007 — Processadores assíncronos e transporte](especificacao/modulos/processadores-assincronos.md) — `refined`
- [MOD-0008 — Ingestão e armazenamento de mídia](especificacao/modulos/ingestao-midia.md) — `refinement`

### Contratos

- [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md) — `refined`
- [CTR-0002 — Execução de script](especificacao/contratos/execucao-script.md) — `refined`
- [CTR-0003 — Job](especificacao/contratos/job.md) — `refined`
- [CTR-0004 — Auditoria e logs](especificacao/contratos/auditoria-logs.md) — `refined`
- [CTR-0005 — Protocolo de processador assíncrono](especificacao/contratos/processador-assincrono.md) — `refined`
- [CTR-0006 — Mídia persistida](especificacao/contratos/midia-persistida.md) — `refined`
- [CTR-0007 — Transmissão persistente de mídia](especificacao/contratos/transmissao-midia.md) — `refined`
- [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](especificacao/contratos/ingestao-midia-messagepack.md) — `refinement`
- [CTR-0009 — Ingestão fracionada de mídia por Unix socket](especificacao/contratos/ingestao-midia-fracionada.md) — `refinement`
- [CTR-0010 — Origem e conversão dos dados persistentes](especificacao/contratos/origem-dados-persistencia.md) — `refinement`

### Fluxos

- [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md) — `refinement`
- [FLW-0002 — Job assíncrono](especificacao/fluxos/job-assincrono.md) — `refinement`
- [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md) — `refinement`
- [FLW-0004 — Encerramento do serviço](especificacao/fluxos/encerramento-servico.md) — `refinement`
- [FLW-0005 — Processamento assíncrono por processador registrado](especificacao/fluxos/processamento-assincrono.md) — `refined`
- [FLW-0006 — Ingestão local de mídia](especificacao/fluxos/ingestao-midia.md) — `refinement`
- [FLW-0007 — Ingestão fracionada de mídia](especificacao/fluxos/ingestao-midia-fracionada.md) — `refinement`
- [FLW-0008 — Limpeza de payloads de mídia transmitida](especificacao/fluxos/limpeza-midia.md) — `refined`
- [FLW-0009 — Registro do estado operacional](especificacao/fluxos/registro-estado-operacional.md) — `refinement`
- [FLW-0010 — Consumo do estado operacional](especificacao/fluxos/consumo-estado-operacional.md) — `refinement`

### Persistência

- [PST-0001 — Proposta de tabelas do estado operacional](especificacao/persistencia/tabelas-estado-operacional.md) — `refinement`
- [PST-0002 — Relacionamentos do estado operacional](especificacao/persistencia/relacionamentos-estado-operacional.md) — `refinement`
- [PST-0003 — Claim concorrente de filas persistentes](especificacao/persistencia/claim-concorrente-filas.md) — `refined`
- [PST-0100 — Entidades persistentes](especificacao/persistencia/entidades/README.md) — `refinement`

### Configuração e requisitos

- [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md) — `refinement`
- [REQ-0001 — Escopo do primeiro MVP](especificacao/requisitos/mvp.md) — `refined`
- [REQ-0002 — Requisitos de testes](especificacao/requisitos/testes.md) — `refined`

## BLOCKED

Os bloqueios e inconsistências que impedem considerar o gate do MVP satisfeito estão consolidados em [Pendências documentais](pendencias.md).

O índice mantém apenas os estados declarados dos documentos; um status `refined` não deve ser interpretado isoladamente como liberação quando o documento de pendências identifica conteúdo insuficiente ou inconsistente.
