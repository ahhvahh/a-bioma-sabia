# Pendências documentais

Este documento consolida as lacunas ainda abertas na documentação do Sabiá. O detalhe normativo permanece nos documentos referenciados.

## Estado do gate

O gate completo de especificação do MVP permanece `BLOCKED` enquanto contratos e especificações necessárias estiverem em `refinement`.

## BLOCKED

### CTR-0001 — Comando interno

- Documento: [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Estado atual do conteúdo: a fronteira conceitual entre adaptadores e Command Router está definida.
- Informação ausente: schema definitivo, tipos concretos, limites de argumentos/anexos e estrutura exata de resultado e erro.
- Dependências afetadas:
  - [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md)
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)
  - [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md)

### CTR-0002 — Execução de script

- Documento: [CTR-0002 — Execução de script](especificacao/contratos/execucao-script.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Estado atual do conteúdo: registro explícito, timeout, stdout, stderr e exit code estão definidos.
- Informação ausente:
  - forma de invocação do executável/script cadastrado;
  - diretório de trabalho;
  - variáveis de ambiente herdadas ou permitidas;
  - limite de stdout e stderr;
  - tratamento de processos filhos em timeout/cancelamento.
- Dependências afetadas:
  - [MOD-0003 — Registro e execução de scripts](especificacao/modulos/scripts.md)
  - [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md)
  - [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md)

## Pendências de refinamento

### Telegram — retry e offset do long polling

- Documentos:
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)
  - [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta definir:
  - quando o offset de `getUpdates` é considerado processado;
  - como o offset necessário ao recovery é persistido;
  - política de retry para falhas da Bot API;
  - comportamento após reinício quando houver update ou resposta pendente.

### Scheduler — sobreposição de execuções

- Documentos:
  - [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md)
  - [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta decidir o comportamento quando um novo disparo da mesma tarefa ocorre enquanto a execução anterior ainda está em andamento.

### Alertas — primeira avaliação, lembrete e falhas

- Documentos:
  - [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md)
  - [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Falta definir:
  - comportamento da primeira avaliação quando não existe estado anterior persistido;
  - política detalhada de lembrete para estado degradado inalterado;
  - comportamento do scheduler quando a própria verificação falha.

## Itens já resolvidos

Não são mais pendências:

- [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md): SQLite e recovery de requisições/respostas pendentes estão `refined`.
- [ADR-0010 — Política de encerramento de jobs](adr/runtime/encerramento-de-jobs.md): aviso aos clientes, espera por respostas e timeout estão `refined`.
- [CTR-0003 — Job](especificacao/contratos/job.md): fila, concorrência, cancelamento, retry e recovery estão `refined`.
- [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md): configuração do MVP está `refined`.

## Condição para liberar o MVP

Antes de iniciar implementação do escopo dependente, os contratos e fluxos necessários devem satisfazer o gate do ADP 1.0. Nenhuma lacuna acima deve ser preenchida por suposição.
