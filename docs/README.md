# Sabiá — documentação técnica

Documentação do projeto **A-Bioma Sabiá**. Os documentos descrevem a solução planejada; a implementação não está disponível nesta branch.

## Guias e referências

| Assunto | Documento |
|---|---|
| Instalação e implantação | [Instalação](instalacao.md) |
| Criar bot Telegram | [Criar bot no Telegram](criar-bot-telegram.md) |
| Interações do operador e do usuário | [Guia do usuário](guia-usuario.md) |
| Arquitetura | [Arquitetura](arquitetura.md) |
| Banco de dados — visão geral | [Persistência](persistencia/readme.md) |
| Interfaces, sockets e serviços | [Interfaces e serviços](interfaces-servicos.md) |
| Contratos técnicos | [Referência de contratos](referencia-contratos.md) |
| Segurança e grupos Linux | [Segurança e acessos](seguranca-acessos.md) |
| Regras consolidadas | [Regras consolidadas](regras-consolidadas-sabia.md) |
| Questões e bloqueios | [Pendências](pendencias.md) |

## Modelo de persistência PostgreSQL

O projeto adota PostgreSQL relacional, com banco proposto `abioma_sabia` e schema `sabia`. O modelo é documental e ainda contém decisões pendentes.

- [Visão geral, banco e schema](persistencia/readme.md)
- [Configuração de banco, UDS, roles e permissões](persistencia/configuracao-banco.md)
- [Relacionamentos e diagrama ER](persistencia/relacionamentos.md)
- [Entidades e campos PostgreSQL](persistencia/entidades/README.md)
- [Concorrência das filas](persistencia/concorrencia.md)
- [Pendências do modelo físico](persistencia/pendencias.md)

## Prefixos documentais

| Prefixo | Categoria | Exemplo real |
|---|---|---|
| `DEC` | Decisão arquitetural (MADR/ADR) | [DEC-001 — Go e binário único](decisoes/DEC-001-go-binario-unico-linux.md) |
| `ARQ` | Contexto, componentes e módulos de arquitetura | [ARQ-001 — Contexto do Sabiá](arquitetura/ARQ-001-contexto-sabia.md) |
| `CONTR` | Contratos técnicos | [CONTR-001 — Comando interno](contratos/CONTR-001-comando-interno.md) |

Os IDs originais `ADR-0001` a `ADR-0013`, `DSG-0001` e `DSG-0002` permanecem como referências históricas; a documentação canônica utiliza prefixos `DEC` e `ARQ`.

## Especificações técnicas recuperadas da `develop`

O índice inclui documentos originalmente referenciados na `develop`. Os arquivos migrados permanecem preservados como legados, e os documentos canônicos passam a concentrar as versões revisadas.

### Decisões

Os documentos `DEC` são versões canônicas revisadas dos ADRs. Os originais em `docs/adr/` permanecem como fontes legadas; decisões históricas conflitantes estão identificadas nos destinos.

- [DEC-001 (ADR-0001) — Go, binário único e serviço Linux](decisoes/DEC-001-go-binario-unico-linux.md) — `refined`
- [DEC-002 (ADR-0002) — Múltiplos clientes Telegram isolados](decisoes/DEC-002-clientes-telegram-isolados.md) — `refined`
- [DEC-003 (ADR-0003) — Core independente do Telegram](decisoes/DEC-003-core-independente-telegram.md) — `refined`
- [DEC-004 (ADR-0004) — Registro explícito de scripts](decisoes/DEC-004-registro-explicito-scripts.md) — `refined`
- [DEC-005 (ADR-0005) — Telegram Long Polling](decisoes/DEC-005-telegram-long-polling.md) — `refined`
- [DEC-006 (ADR-0006) — Jobs assíncronos](decisoes/DEC-006-jobs-assincronos.md) — `refined`
- [DEC-007 (ADR-0007) — Scheduler e alertas orientados a estado](decisoes/DEC-007-alertas-estado.md) — `refined`
- [DEC-008 (ADR-0008) — Menor privilégio e autorização explícita](decisoes/DEC-008-menor-privilegio-autorizacao.md) — `refined`
- [DEC-009 (ADR-0009) — Persistência do estado operacional](decisoes/DEC-009-postgresql-estado-operacional.md) — `refined`
- [DEC-010 (ADR-0010) — Política de encerramento de jobs](decisoes/DEC-010-encerramento-jobs.md) — `refined`
- [DEC-011 (ADR-0011) — Processadores assíncronos registrados e transporte de progresso](decisoes/DEC-011-processadores-assincronos-registrados.md) — `refined`
- [DEC-012 (ADR-0012) — Ingestão persistente de mídia por Unix socket](decisoes/DEC-012-ingestao-midia-socket-messagepack.md) — `refined`
- [DEC-013 (ADR-0013) — Dois canais de ingestão de mídia e upload fracionado](decisoes/DEC-013-ingestao-midia-fracionada.md) — `refined`

### Desenhos de arquitetura

Os arquivos `ARQ` concentram os desenhos canônicos; os `DSG` originais permanecem preservados em `docs/desenho/`.

- [ARQ-001 (DSG-0001) — Contexto do Sabiá](arquitetura/ARQ-001-contexto-sabia.md) — `finalized`
- [ARQ-002 (DSG-0002) — Componentes do Sabiá Core](arquitetura/ARQ-002-componentes-core.md) — `finalized`

### Módulos

As especificações de módulos migradas estão em `docs/arquitetura/`; as demais continuam em `docs/especificacao/modulos/` até revisão posterior.

- [ARQ-003 (MOD-0001) — Core e Command Router](arquitetura/ARQ-003-core-command-router.md) — `refined`
- [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md) — `refinement`
- [ARQ-004 (MOD-0003) — Registro e execução de scripts](arquitetura/ARQ-004-registro-execucao-scripts.md) — `refined`
- [ARQ-005 (MOD-0004) — Jobs](arquitetura/ARQ-005-jobs.md) — `refined`
- [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md) — `refinement`
- [ARQ-006 (MOD-0006) — Segurança e autorização](arquitetura/ARQ-006-seguranca-autorizacao.md) — `refined`
- [ARQ-007 (MOD-0007) — Processadores assíncronos e transporte](arquitetura/ARQ-007-processadores-assincronos.md) — `refined`
- [MOD-0008 — Ingestão e armazenamento de mídia](especificacao/modulos/ingestao-midia.md) — `refinement`

### Contratos

Os [contratos canônicos](contratos/README.md) foram revisados documentalmente. Os registros `CTR` originais permanecem preservados; contratos em `refinement` e cláusulas `BLOCKED` ainda exigem decisões técnicas.

- [CONTR-001 (CTR-0001) — Comando interno](contratos/CONTR-001-comando-interno.md) — `refined`
- [CONTR-002 (CTR-0002) — Execução de script](contratos/CONTR-002-execucao-script.md) — `refined`
- [CONTR-003 (CTR-0003) — Job](contratos/CONTR-003-job.md) — `refined`
- [CONTR-004 (CTR-0004) — Auditoria e logs](contratos/CONTR-004-auditoria-logs.md) — `refinement`
- [CONTR-005 (CTR-0005) — Protocolo de processador assíncrono](contratos/CONTR-005-processador-assincrono.md) — `refined`
- [CONTR-006 (CTR-0006) — Mídia persistida](contratos/CONTR-006-midia-persistida.md) — `refined`
- [CONTR-007 (CTR-0007) — Transmissão persistente de mídia](contratos/CONTR-007-transmissao-midia.md) — `refined`
- [CONTR-008 (CTR-0008) — Ingestão de mídia por Unix socket e MessagePack](contratos/CONTR-008-ingestao-midia-messagepack.md) — `refinement`
- [CONTR-009 (CTR-0009) — Ingestão fracionada de mídia por Unix socket](contratos/CONTR-009-ingestao-midia-fracionada.md) — `refinement`
- [CONTR-010 (CTR-0010) — Origem e conversão dos dados persistentes](contratos/CONTR-010-origem-dados-persistencia.md) — `refinement`

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

**Nota:** os arquivos de persistência já recuperados estão em [Tabelas e modelo PostgreSQL](#modelo-de-persistência-postgresql), sem cópias redundantes. Referências internas dos documentos originais a arquivos fora deste conjunto podem continuar apontando para conteúdo ainda não recuperado.
