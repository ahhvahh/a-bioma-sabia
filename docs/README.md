# Sabiá — documentação técnica

Esta é a raiz documental do Sabiá, organizada conforme o Architecture Documentation Pipeline (ADP) 1.0.

O Sabiá é um serviço Linux do ecossistema Bioma para integrar aplicações, scripts e serviços locais com clientes Telegram independentes, com foco em segurança, baixo consumo, modularidade e operação sem execução arbitrária de comandos.

## Estado do pipeline

A arquitetura principal possui decisões explícitas e desenhos finalizados. O desenvolvimento completo do MVP ainda depende dos contratos indicados em `BLOCKED`.

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

### Desenhos

- [DSG-0001 — Contexto do Sabiá](desenho/contexto-sabia.md) — `finalized`
- [DSG-0002 — Componentes do Sabiá Core](desenho/componentes-core.md) — `finalized`

### Módulos

- [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md) — `refinement`
- [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md) — `refinement`
- [MOD-0003 — Registro e execução de scripts](especificacao/modulos/scripts.md) — `refinement`
- [MOD-0004 — Jobs](especificacao/modulos/jobs.md) — `refined`
- [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md) — `refinement`
- [MOD-0006 — Segurança e autorização](especificacao/modulos/seguranca.md) — `refined`

### Contratos

- [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md) — `refinement`
- [CTR-0002 — Execução de script](especificacao/contratos/execucao-script.md) — `refinement`
- [CTR-0003 — Job](especificacao/contratos/job.md) — `refined`
- [CTR-0004 — Auditoria e logs](especificacao/contratos/auditoria-logs.md) — `refined`

### Fluxos

- [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md) — `refinement`
- [FLW-0002 — Job assíncrono](especificacao/fluxos/job-assincrono.md) — `refined`
- [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md) — `refinement`
- [FLW-0004 — Encerramento do serviço](especificacao/fluxos/encerramento-servico.md) — `refinement`

### Configuração e requisitos

- [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md) — `refined`
- [REQ-0001 — Escopo do primeiro MVP](especificacao/requisitos/mvp.md) — `refined`
- [REQ-0002 — Requisitos de testes](especificacao/requisitos/testes.md) — `refined`

## BLOCKED

### Contrato do comando interno

- Documento: [CTR-0001](especificacao/contratos/comando-interno.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta fechar schema, tipos, limites, anexos e estrutura exata de resultado/erro.
- Afeta: MOD-0001, MOD-0002 e FLW-0001.

### Contrato operacional do executor

- Documento: [CTR-0002](especificacao/contratos/execucao-script.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta definir invocação do processo, diretório de trabalho, ambiente permitido, limites de saída e tratamento de processos no timeout.
- Afeta: MOD-0003, MOD-0005 e FLW-0003.

## Pendências não bloqueadoras isoladas

- fechar política de offset/retry do Telegram para MOD-0002;
- fechar concorrência de duas execuções do mesmo agendamento;
- definir comportamento da primeira avaliação de alerta quando ainda não existe estado anterior.

Nenhuma dessas lacunas deve ser preenchida por suposição.
