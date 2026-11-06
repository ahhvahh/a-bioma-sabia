# Sabiá — documentação técnica

Documentação do projeto **A-Bioma Sabiá**. Os documentos descrevem a solução planejada; a implementação não está disponível nesta branch.

## Guias e referências

Os [guias organizados](guias/README.md) descrevem procedimentos planejados, não instalação validada. A documentação de [persistência](persistencia/readme.md) identifica explicitamente os modelos propostos.

| Assunto | Documento |
|---|---|
| Instalação e implantação | [Instalação](guias/GUIA-001-preparar-instalacao.md) |
| Criar bot Telegram | [Criar bot no Telegram](guias/GUIA-002-criar-bot-telegram.md) |
| Interações do operador e do usuário | [Guia do usuário](guia-usuario.md) |
| Arquitetura | [Arquitetura](arquitetura.md) |
| Banco de dados — visão geral | [Persistência](persistencia/readme.md) |
| Interfaces, sockets e serviços | [Interfaces e serviços](arquitetura/ARQ-011-inventario-canais-servicos.md) |
| Contratos técnicos | [Referência de contratos](referencia-contratos.md) |
| Segurança e grupos Linux | [Segurança e acessos](arquitetura/ARQ-006-seguranca-autorizacao.md) |
| Regras consolidadas | [Rastreabilidade das regras](referencia/REF-005-rastreabilidade-regras.md) · [original legado](regras-consolidadas-sabia.md) |
| Questões e bloqueios | [Pendências](referencia/REF-006-gate-mvp.md) |

## Absorção das regras consolidadas

- [REF-005 — Mapeamento das 14 seções](referencia/REF-005-rastreabilidade-regras.md)
- [CONTR-011 — API administrativa UDS](contratos/CONTR-011-api-administrativa-uds.md)
- [CONTR-012 — Assinatura por eventos](contratos/CONTR-012-assinaturas-eventos.md)
- [FLUX-011 — Aprovação de acesso Telegram](fluxos/FLUX-011-aprovacao-acesso-telegram.md)
- [FLUX-012 — Validação e publicação de scripts](fluxos/FLUX-012-validacao-publicacao-script.md)

A absorção histórica foi documentada; os novos contratos e fluxos continuam em `refinement`, e o gate do MVP permanece `BLOCKED`.

## Interfaces administrativas

- [Inventário de canais e serviços](arquitetura/ARQ-011-inventario-canais-servicos.md)
- [Terminal administrativo `sabiactl`](interfaces/terminal/INTF-TERM-001-sabiactl.md)
- [Índice de interfaces](interfaces/README.md)

## Modelo de persistência PostgreSQL

O projeto adota PostgreSQL relacional, com banco proposto `abioma_sabia` e schema `sabia`. O modelo é documental e ainda contém decisões pendentes.

- [Visão geral, banco e schema](persistencia/readme.md)
- [Configuração de banco, UDS, roles e permissões](persistencia/configuracao-banco.md)
- [Relacionamentos e diagrama ER](persistencia/PERS-001-relacionamentos.md)
- [Entidades e campos PostgreSQL](persistencia/entidades/README.md) — 16 fichas canônicas `ENT-001` a `ENT-016`; dez operacionais, uma condicional e cinco administrativas propostas
- [Concorrência das filas](persistencia/concorrencia.md)
- [Pendências do modelo físico](persistencia/pendencias.md)

## Prefixos documentais

| Prefixo | Categoria | Exemplo real |
|---|---|---|
| `DEC` | Decisão arquitetural (MADR/ADR) | [DEC-001 — Go e binário único](decisoes/DEC-001-go-binario-unico-linux.md) |
| `ARQ` | Contexto, componentes e módulos de arquitetura | [ARQ-001 — Contexto do Sabiá](arquitetura/ARQ-001-contexto-sabia.md) |
| `CONTR` | Contratos técnicos | [CONTR-001 — Comando interno](contratos/CONTR-001-comando-interno.md) |
| `FLUX` | Fluxos de execução e sequência | [FLUX-001 — Comando Telegram](fluxos/FLUX-001-comando-telegram.md) |
| `REF` | Referências de configuração, escopo e testes | [REF-001 — Modelo de configuração](referencia/REF-001-modelo-configuracao.md) |
| `GUIA` | Procedimentos de preparação e operação | [GUIA-001 — Preparar instalação](guias/GUIA-001-preparar-instalacao.md) |
| `PERS` | Modelos e relacionamentos de dados | [PERS-001 — Relacionamentos PostgreSQL](persistencia/PERS-001-relacionamentos.md) |
| `ENT` | Fichas de entidades e campos SQL | [ENT-001 — transport_cursor](persistencia/entidades/ENT-001-transport-cursor.md) |
| `INTF-TERM` | Interface administrativa de terminal | [INTF-TERM-001 — sabiactl](interfaces/terminal/INTF-TERM-001-sabiactl.md) |

Os IDs originais `ADR-0001` a `ADR-0013`, `DSG-0001` a `DSG-0002`, `MOD-0001` a `MOD-0008`, `CFG-0001`, `REQ-0001` a `REQ-0002` e os identificadores `PST` permanecem rastreáveis. Os novos documentos usam `DEC`, `ARQ`, `CONTR`, `FLUX`, `REF`, `GUIA` e `PERS` conforme o assunto.

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

As oito especificações históricas `MOD` foram incorporadas aos documentos canônicos em `docs/arquitetura/`; os originais permanecem disponíveis para rastreabilidade.

- [ARQ-003 (MOD-0001) — Core e Command Router](arquitetura/ARQ-003-core-command-router.md) — `refined`
- [ARQ-008 (MOD-0002) — Adaptador Telegram](arquitetura/ARQ-008-adaptador-telegram.md) — `refinement`
- [ARQ-004 (MOD-0003) — Registro e execução de scripts](arquitetura/ARQ-004-registro-execucao-scripts.md) — `refined`
- [ARQ-005 (MOD-0004) — Jobs](arquitetura/ARQ-005-jobs.md) — `refined`
- [ARQ-009 (MOD-0005) — Scheduler e Alert Manager](arquitetura/ARQ-009-scheduler-alertas.md) — `refinement`
- [ARQ-006 (MOD-0006) — Segurança e autorização](arquitetura/ARQ-006-seguranca-autorizacao.md) — `refined`
- [ARQ-007 (MOD-0007) — Processadores assíncronos e transporte](arquitetura/ARQ-007-processadores-assincronos.md) — `refined`
- [ARQ-010 (MOD-0008) — Ingestão e armazenamento de mídia](arquitetura/ARQ-010-ingestao-midia.md) — `refinement`

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

Os [fluxos canônicos](fluxos/README.md) contêm diagramas de sequência e preservam os identificadores originais `FLW`. A revisão documental não comprova implementação, nem encerra estados `refinement` ou `BLOCKED`.

- [FLUX-001 (FLW-0001) — Comando Telegram](fluxos/FLUX-001-comando-telegram.md) — `refinement`
- [FLUX-002 (FLW-0002) — Job assíncrono](fluxos/FLUX-002-job-assincrono.md) — `refinement`
- [FLUX-003 (FLW-0003) — Monitoramento agendado](fluxos/FLUX-003-monitoramento-agendado.md) — `refinement`
- [FLUX-004 (FLW-0004) — Encerramento do serviço](fluxos/FLUX-004-encerramento-servico.md) — `refined`
- [FLUX-005 (FLW-0005) — Processamento assíncrono por processador registrado](fluxos/FLUX-005-processamento-assincrono.md) — `refined`
- [FLUX-006 (FLW-0006) — Ingestão local de mídia](fluxos/FLUX-006-ingestao-midia.md) — `refinement`
- [FLUX-007 (FLW-0007) — Ingestão fracionada de mídia](fluxos/FLUX-007-ingestao-midia-fracionada.md) — `refinement`
- [FLUX-008 (FLW-0008) — Limpeza de payloads de mídia transmitida](fluxos/FLUX-008-limpeza-midia.md) — `refined`
- [FLUX-009 (FLW-0009) — Registro do estado operacional](fluxos/FLUX-009-registro-estado-operacional.md) — `refinement`
- [FLUX-010 (FLW-0010) — Consumo do estado operacional](fluxos/FLUX-010-consumo-estado-operacional.md) — `refinement`

### Configuração e requisitos

Os documentos canônicos correspondentes estão no [índice de referências técnicas](referencia/README.md). Configurações em `refinement` ainda contêm decisões `BLOCKED`.

- [REF-001 (CFG-0001) — Modelo de configuração](referencia/REF-001-modelo-configuracao.md) — `refinement`
- [REF-002 (REQ-0001) — Escopo do primeiro MVP](referencia/REF-002-escopo-primeiro-mvp.md) — `refined`
- [REF-003 (REQ-0002) — Requisitos de testes](referencia/REF-003-requisitos-testes.md) — `refined`

## Revisão Telegram 2026-10-21

- [Contrato operacional e decisões de integração](referencia/REF-004-operacao-telegram.md)
- [Caderno de testes locais organizado por grupos](testes/telegram/README.md)

**Nota:** os arquivos de persistência já recuperados estão em [Tabelas e modelo PostgreSQL](#modelo-de-persistência-postgresql), sem cópias redundantes. Referências internas dos documentos originais a arquivos fora deste conjunto podem continuar apontando para conteúdo ainda não recuperado.
