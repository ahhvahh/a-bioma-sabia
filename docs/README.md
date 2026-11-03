# Sabiá — documentação técnica

Documentação do projeto **A-Bioma Sabiá**. Os documentos descrevem a solução planejada; a implementação não está disponível nesta branch.

## Guias e referências

| Assunto | Documento |
|---|---|
| Instalação e implantação | [Instalação](instalacao.md) |
| Criar bot Telegram | [Criar bot no Telegram](criar-bot-telegram.md) |
| Interações do operador e do usuário | [Guia do usuário](guia-usuario.md) |
| Arquitetura | [Arquitetura](arquitetura.md) |
| Banco de dados — visão geral | [Persistência](persistencia.md) |
| Interfaces, sockets e serviços | [Interfaces e serviços](interfaces-servicos.md) |
| Contratos técnicos | [Referência de contratos](referencia-contratos.md) |
| Segurança e grupos Linux | [Segurança e acessos](seguranca-acessos.md) |
| Regras consolidadas | [Regras consolidadas](regras-consolidadas-sabia.md) |
| Questões e bloqueios | [Pendências](pendencias.md) |

## Tabelas e modelo PostgreSQL

Os documentos de [`/docs/presistencia`](presistencia/README.md) foram recuperados da branch `develop`. **Os estados de refinamento de cada documento foram mantidos**; propostas em `refinement` não estão automaticamente aprovadas.

| Documento | Descrição |
|---|---|
| [Proposta de tabelas — PST-0001](presistencia/tabelas-estado-operacional.md) | `tabelas-estado-operacional.md` |
| [Relacionamentos — PST-0002](presistencia/relacionamentos-estado-operacional.md) | `relacionamentos-estado-operacional.md` |
| [Concorrência nas filas — PST-0003](presistencia/claim-concorrente-filas.md) | `claim-concorrente-filas.md` |
| [transport_cursor — PST-0101](presistencia/transport-cursor.md) | `transport-cursor.md` |
| [inbound_update — PST-0102](presistencia/inbound-update.md) | `inbound-update.md` |
| [request — PST-0103](presistencia/request.md) | `request.md` |
| [job — PST-0104](presistencia/job.md) | `job.md` |
| [job_state_history — PST-0105](presistencia/job-state-history.md) | `job-state-history.md` |
| [outbound_message — PST-0106](presistencia/outbound-message.md) | `outbound-message.md` |
| [alert_state — PST-0107](presistencia/alert-state.md) | `alert-state.md` |
| [media — PST-0108](presistencia/media.md) | `media.md` |
| [media_chunk — PST-0109](presistencia/media-chunk.md) | `media-chunk.md` |
| [media_transmission — PST-0110](presistencia/media-transmission.md) | `media-transmission.md` |
| [schema_version — PST-0111](presistencia/schema-version.md) | `schema-version.md` |

**Referência:** [índice das entidades persistentes](presistencia/README.md). Documentos e decisões anteriores ainda disponíveis no histórico de `develop`.

## Especificações técnicas recuperadas da `develop`

Arquivos restaurados **integralmente**, com seus caminhos originais e estados documentais preservados. O índice inclui somente os documentos diretamente referenciados no `docs/README.md` da branch `develop`.

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

- [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md) — `refined`
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
- [CTR-0004 — Auditoria e logs](especificacao/contratos/auditoria-logs.md) — `refinement`
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
- [FLW-0004 — Encerramento do serviço](especificacao/fluxos/encerramento-servico.md) — `refined`
- [FLW-0005 — Processamento assíncrono por processador registrado](especificacao/fluxos/processamento-assincrono.md) — `refined`
- [FLW-0006 — Ingestão local de mídia](especificacao/fluxos/ingestao-midia.md) — `refinement`
- [FLW-0007 — Ingestão fracionada de mídia](especificacao/fluxos/ingestao-midia-fracionada.md) — `refinement`
- [FLW-0008 — Limpeza de payloads de mídia transmitida](especificacao/fluxos/limpeza-midia.md) — `refined`
- [FLW-0009 — Registro do estado operacional](especificacao/fluxos/registro-estado-operacional.md) — `refinement`
- [FLW-0010 — Consumo do estado operacional](especificacao/fluxos/consumo-estado-operacional.md) — `refinement`

### Configuração e requisitos

- [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md) — `refinement`
- [REQ-0001 — Escopo do primeiro MVP](especificacao/requisitos/mvp.md) — `refined`
- [REQ-0002 — Requisitos de testes](especificacao/requisitos/testes.md) — `refined`

## Revisão Telegram 2026-10-21

- [Contrato operacional e decisões de integração](telegram-operacao-2026.md)
- [Caderno de testes locais organizado por grupos](testes/telegram/README.md)

**Nota:** os arquivos de persistência já recuperados estão em [Tabelas e modelo PostgreSQL](#tabelas-e-modelo-postgresql), sem cópias redundantes. Referências internas dos documentos originais a arquivos fora deste conjunto podem continuar apontando para conteúdo ainda não recuperado.
