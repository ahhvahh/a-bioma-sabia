<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# DEC-006 — Jobs assíncronos

**ID:** `DEC-006`  
**ID histórico:** `ADR-0006`  
**Estado documental de origem:** `refined`  
**Versão histórica registrada:** `1`  
**Implementação:** não verificada nesta revisão

## Decisão

Operações demoradas serão representadas por jobs assíncronos processados por fila e workers. Uma mesma `request` pode criar zero ou vários jobs. No MVP, a quantidade máxima de jobs simultaneamente em `running` é igual à quantidade de threads lógicas de CPU disponíveis ao processo no startup; jobs excedentes permanecem `queued`.

Estados previstos: `queued`, `running`, `completed`, `failed`, `cancelled` e `timeout`.

## Contexto e problema

Downloads, conversões e outras operações demoradas não podem bloquear o recebimento de novos comandos.

Definir como o Sabiá executa trabalho demorado.

## Restrições

- resposta ao usuário deve ser rápida;
- progresso deve poder atualizar a mensagem do job;
- o resultado final pode incluir arquivo;
- são necessários JobManager, JobQueue e Worker.

## Alternativas consideradas

### Execução síncrona no tratamento do comando

Bloqueia a capacidade de receber novos comandos.

### Job assíncrono

Separa aceitação do comando e execução demorada.

## Justificativa

Mantém o canal responsivo e permite observabilidade de progresso.

## Consequências

- o sistema precisa correlacionar job e destino de resposta;
- chat, mensagem e job precisam ser correlacionáveis para atualização de progresso;
- persistência, retry e recovery são detalhados nas especificações dependentes; a concorrência do MVP é limitada pela quantidade de threads lógicas de CPU disponíveis ao processo no startup.

## Critérios de validação

- um job demorado não bloqueia o processamento de outro comando;
- a quantidade de jobs simultaneamente em `running` não excede as threads lógicas de CPU disponíveis ao processo;
- uma mesma request pode originar vários jobs sem unicidade em `job.request_id`;
- o estado do job evolui por estados válidos;
- progresso pode ser refletido na mensagem associada.

## Dependências

- [ADR-0003 — Core independente do Telegram](DEC-003-core-independente-telegram.md)

## Limitações da revisão

Uma mesma `request` pode gerar vários jobs; a quantidade máxima de jobs em execução foi definida na origem em função das threads lógicas de CPU no startup. O esquema físico está na [entidade job](../persistencia/entidades/ENT-004-job.md), sem comprovação de testes de concorrência.

A revisão documental não demonstra implementação, execução de testes ou aprovação de novas regras.

## Fonte e rastreabilidade

Documento histórico preservado: [`ADR-0006` original](https://github.com/ahhvahh/a-bioma-sabia/blob/f898239c4519b1c8fd21327adf7a8407a1bbeaf7/docs/adr/processamento/jobs-assincronos.md). O identificador e a versão original foram mantidos para rastrear a decisão, sem renumerar outros ADRs.
