# Sabiá — documentação técnica

Esta é a raiz documental do Sabiá, organizada conforme o Architecture Documentation Pipeline (ADP) 1.0.

O Sabiá é um serviço Linux do ecossistema Bioma para integrar aplicações, scripts e serviços locais com clientes Telegram independentes, com foco em segurança, baixo consumo, modularidade e operação sem execução arbitrária de comandos.

## Estado do pipeline

A arquitetura principal possui decisões explícitas e desenhos finalizados. A especificação do MVP ainda possui itens em `refinement` porque existem decisões operacionais não fechadas.

### Decisões

- [ADR-0001 — Go, binário único e serviço Linux](adr/runtime/go-binario-unico.md) — `refined`
- [ADR-0002 — Múltiplos clientes Telegram isolados](adr/telegram/multiplos-clientes-isolados.md) — `refined`
- [ADR-0003 — Core independente do Telegram](adr/arquitetura/core-independente-do-telegram.md) — `refined`
- [ADR-0004 — Registro explícito de scripts](adr/execucao/registro-explicito-de-scripts.md) — `refined`
- [ADR-0005 — Telegram Long Polling](adr/telegram/long-polling.md) — `refined`
- [ADR-0006 — Jobs assíncronos](adr/processamento/jobs-assincronos.md) — `refined`
- [ADR-0007 — Scheduler e alertas orientados a estado](adr/monitoramento/scheduler-alertas-estado.md) — `refined`
- [ADR-0008 — Menor privilégio e autorização explícita](adr/seguranca/menor-privilegio-e-autorizacao.md) — `refined`
- [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md) — `refinement`
- [ADR-0010 — Política de encerramento de jobs](adr/runtime/encerramento-de-jobs.md) — `refinement`

### Desenhos

- [DSG-0001 — Contexto do Sabiá](desenho/contexto-sabia.md) — `finalized`
- [DSG-0002 — Componentes do Sabiá Core](desenho/componentes-core.md) — `finalized`

### Módulos

- [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md) — `refinement`
- [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md) — `refinement`
- [MOD-0003 — Registro e execução de scripts](especificacao/modulos/scripts.md) — `refinement`
- [MOD-0004 — Jobs](especificacao/modulos/jobs.md) — `refinement`
- [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md) — `refinement`
- [MOD-0006 — Segurança e autorização](especificacao/modulos/seguranca.md) — `refined`

### Contratos

- [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md) — `refinement`
- [CTR-0002 — Execução de script](especificacao/contratos/execucao-script.md) — `refinement`
- [CTR-0003 — Job](especificacao/contratos/job.md) — `refinement`
- [CTR-0004 — Auditoria e logs](especificacao/contratos/auditoria-logs.md) — `refined`

### Fluxos

- [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md) — `refinement`
- [FLW-0002 — Job assíncrono](especificacao/fluxos/job-assincrono.md) — `refinement`
- [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md) — `refinement`
- [FLW-0004 — Encerramento do serviço](especificacao/fluxos/encerramento-servico.md) — `refinement`

### Configuração e requisitos

- [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md) — `refinement`
- [REQ-0001 — Escopo do primeiro MVP](especificacao/requisitos/mvp.md) — `refined`
- [REQ-0002 — Requisitos de testes](especificacao/requisitos/testes.md) — `refined`

## BLOCKED

### Persistência do estado operacional

- Documento: [ADR-0009](adr/persistencia/estado-operacional.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta decidir onde e como persistir jobs, fila, offsets relevantes do Telegram e estado dos alertas entre reinicializações.
- Afeta: MOD-0002, MOD-0004, MOD-0005, CTR-0003, FLW-0002 e FLW-0003.

### Encerramento de jobs em execução

- Documento: [ADR-0010](adr/runtime/encerramento-de-jobs.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta decidir se jobs em execução serão concluídos, cancelados ou tratados por política configurável ao receber SIGTERM/SIGINT.
- Afeta: MOD-0004 e FLW-0004.

### Contrato operacional do executor

- Documento: [CTR-0002](especificacao/contratos/execucao-script.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta definir detalhes necessários à execução segura: forma de invocação do arquivo cadastrado, diretório de trabalho, ambiente herdado/permitido e limites de saída.

### Fila e concorrência de jobs

- Documento: [CTR-0003](especificacao/contratos/job.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta definir concorrência, capacidade da fila, política de cancelamento, retries e comportamento após reinício.

Nenhum desses bloqueios autoriza preencher a decisão por suposição.
