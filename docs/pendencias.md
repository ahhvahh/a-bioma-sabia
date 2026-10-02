# Pendências documentais

Este documento consolida as lacunas ainda abertas na documentação do Sabiá. O detalhe normativo permanece nos documentos referenciados.

## Estado do gate

O gate completo de especificação do MVP permanece `BLOCKED`.

Além de documentos ainda em `refinement`, existem documentos marcados como `refined` cujo conteúdo ainda não é suficiente para implementação sem decisões implícitas. Esses casos precisam ser corrigidos antes de considerar o gate satisfeito.

## BLOCKED

### CTR-0001 — Comando interno

- Documento: [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md)
- Estado atual: `refinement`
- Estado necessário: `refined`
- Estado atual do conteúdo:
  - a fronteira conceitual entre adaptadores e Command Router está definida;
  - argumentos são entregues ao Core como `string[]` já tokenizado;
  - tokenização pertence ao adaptador;
  - validação semântica dos argumentos pertence à operação correspondente;
  - identidade e correlação estão definidas por `request_id`, `client_id`, `principal_id`, `reply_context` e `received_at`;
  - `reply_context` contém apenas `transport` e `destination_id`, mantendo o Core independente do Telegram;
  - saída usa envelope discriminado `message | job | file | error`;
  - `message` possui `text`, `job` possui `job_id` e `error` possui `code`, `message` e `retryable`.
- Estado atual adicional:
  - anexos de entrada usam `{name, path}`;
  - resultado `file` usa `{name, path}`;
  - arquivos ficam em `/tmp/sabia/media/<request_id>/`;
  - múltiplos arquivos assíncronos são publicados por múltiplos eventos `content`.
- Informação ausente:
  - lifecycle/limpeza dos arquivos temporários;
  - determinação do tipo de mídia;
  - limites máximos de arquivo.
- Dependências afetadas:
  - [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md)
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)
  - [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md)

### Processadores assíncronos e transporte de mídia

- Decisão relacionada: [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](adr/processamento/processadores-assincronos-registrados.md)
- Documentos:
  - [MOD-0007 — Processadores assíncronos e transporte](especificacao/modulos/processadores-assincronos.md)
  - [CTR-0005 — Protocolo de processador assíncrono](especificacao/contratos/processador-assincrono.md)
  - [FLW-0005 — Processamento assíncrono por processador registrado](especificacao/fluxos/processamento-assincrono.md)
  - [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md)
- Estado atual: ADR em `refined`; módulo, contrato, fluxo e configuração em `refinement`.
- Estado necessário: especificações implementáveis em `refined`.
- Decisões já fechadas:
  - processadores podem ser script Bash, aplicação/executável ou serviço/socket;
  - toda execução usa `request_id` gerado pelo Sabiá;
  - eventos de processador usam `status: loading | content | finally`;
  - `loading` representa progresso e não encerra o job;
  - `content` referencia um arquivo como `{name, path}`, pode ocorrer múltiplas vezes e não encerra o job;
  - `finally` é a finalização semântica normal e contém a mensagem final;
  - binários não trafegam no protocolo de controle;
  - a raiz temporária é `/tmp/sabia/media`, isolada por `request_id`;
  - o Sabiá faz streaming dos arquivos ao Telegram;
  - término do processo sem `finally` não equivale automaticamente a sucesso;
  - imagens e vídeos são suportados na entrada e na saída por referência local.
- Informação ausente:
  - framing/protocolo concreto usado por scripts/aplicações locais;
  - framing/protocolo concreto usado por serviços/socket;
  - política de limpeza dos arquivos/diretórios temporários;
  - comportamento após restart/reboot quando a mídia temporária desaparecer;
  - regra para determinar se o arquivo será enviado como imagem, vídeo ou documento;
  - limites máximos de arquivo;
  - schema da configuração `processors`.
- Dependências afetadas:
  - [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md)
  - [MOD-0004 — Jobs](especificacao/modulos/jobs.md)
  - [FLW-0002 — Job assíncrono](especificacao/fluxos/job-assincrono.md)
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)

### Persistência operacional — especificação técnica ausente

- Decisão relacionada: [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md)
- Estado atual: decisão arquitetural `refined`, mas não existe especificação técnica de persistência.
- Estado necessário: especificação de persistência `refined`.
- Informação ausente:
  - entidades/tabelas e relações;
  - campos, tipos, chaves e restrições;
  - representação de requisições, jobs, tentativas, respostas pendentes, entregas e estado de alertas;
  - regras de atomicidade e idempotência necessárias ao recovery;
  - versionamento/migração do schema em nível implementável.
- Dependências afetadas:
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)
  - [MOD-0004 — Jobs](especificacao/modulos/jobs.md)
  - [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md)
  - [CTR-0003 — Job](especificacao/contratos/job.md)
  - [FLW-0002 — Job assíncrono](especificacao/fluxos/job-assincrono.md)
  - [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md)
  - [FLW-0004 — Encerramento do serviço](especificacao/fluxos/encerramento-servico.md)
  - [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md)

### Scheduler e Alert Manager — semântica operacional incompleta

- Documentos:
  - [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md)
  - [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md)
  - [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md)
- Estado atual: módulo, fluxo e CFG-0001 em `refinement`; além de `schedules`, CFG-0001 foi reaberto pela inclusão de `processors`.
- Estado necessário: especificações implementáveis em `refined`.
- Informação ausente:
  - campos, tipos e defaults necessários da configuração de agendamento.
- Decisões já fechadas:
  - se a mesma tarefa vencer enquanto sua execução anterior estiver ativa, o novo disparo é ignorado e registrado; não há fila nem execução concorrente;
  - na primeira avaliação, `OK` é persistido sem alerta; `WARNING`, `CRITICAL` e `UNKNOWN` geram alerta imediato e são persistidos;
  - falha técnica da verificação ou timeout produz `UNKNOWN`, preserva o motivo técnico em observabilidade e segue as mesmas regras de alerta/persistência dos demais estados;
  - `reminder_interval` é opcional por agendamento; sem ele não há lembrete, com ele estados `WARNING`, `CRITICAL` e `UNKNOWN` inalterados são lembrados somente após o intervalo; mudança de estado reinicia a contagem e `OK` encerra lembretes;
  - a periodicidade do MVP usa somente `interval` de duração positivo e obrigatório; cron, calendários e horários absolutos ficam fora do MVP.

### Comandos do MVP — contrato operacional ausente

- Documento de escopo: [REQ-0001 — Escopo do primeiro MVP](especificacao/requisitos/mvp.md)
- Estado atual: o requisito lista comandos, mas não existe especificação que defina o contrato operacional de cada comando.
- Estado necessário: especificação correspondente `refined`.
- Informação ausente:
  - sintaxe e argumentos aceitos;
  - vínculo entre comando e operação/script cadastrado;
  - validações;
  - resposta de sucesso;
  - erros controlados;
  - comportamento específico de comandos como `/run`, `/jobs` e `/job <id>`.
- Dependências afetadas:
  - [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md)
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)
  - [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md)

### CTR-0004 — Auditoria e logs

- Documento: [CTR-0004 — Auditoria e logs](especificacao/contratos/auditoria-logs.md)
- Estado atual: `refined`, incompatível com o nível de definição atual do contrato.
- Estado necessário: permanecer `refined` somente após completar o contrato.
- Informação ausente:
  - tipos dos campos;
  - campos obrigatórios por tipo de evento;
  - estrutura estável dos registros;
  - identificação dos eventos auditáveis e respectivos resultados;
  - tratamento normativo de dados potencialmente sensíveis além da lista de segredos proibidos.
- Dependências afetadas:
  - [MOD-0006 — Segurança e autorização](especificacao/modulos/seguranca.md)
  - requisito de logs do [REQ-0001 — MVP](especificacao/requisitos/mvp.md)

## Pendências condicionais

### CTR-0003 — modelo de retry

- Documento: [CTR-0003 — Job](especificacao/contratos/job.md)
- Estado atual: `refined`.
- Ponto inconsistente: o contrato declara estados finais imutáveis e, ao mesmo tempo, permite retry como nova tentativa associada ao mesmo job, sem definir o modelo da tentativa.
- Falta definir, se retry automático fizer parte do escopo implementado:
  - identidade e estado da tentativa;
  - quais falhas permitem retry;
  - relação entre estado do job e estado das tentativas;
  - momento em que `max_retries` é consumido.
- Não bloqueia o MVP enquanto `max_retries = 0` e retry automático não for ativado.

## Inconsistências documentais a corrigir

### ADR-0006 — informação desatualizada

[ADR-0006 — Jobs assíncronos](adr/processamento/jobs-assincronos.md) ainda afirma que persistência, concorrência, retries e recovery precisam ser fechados em especificação. Parte desses pontos já foi definida em ADR-0009, ADR-0010 e CTR-0003. O texto precisa ser reconciliado sem alterar a decisão arquitetural.

### ADR-0007 — persistência de alertas desatualizada

[ADR-0007 — Scheduler e alertas orientados a estado](adr/monitoramento/scheduler-alertas-estado.md) ainda afirma que a persistência do estado entre reinícios não está decidida. ADR-0009 já definiu SQLite e recovery desse estado.

### FLW-0004 — estado em refinement sem lacuna própria explícita

[FLW-0004 — Encerramento do serviço](especificacao/fluxos/encerramento-servico.md) está em `refinement`, embora o fluxo já detalhe gatilho, sequência, timeout, falhas e recovery. Antes de mudar o status, deve ser verificado se a dependência em MOD-0005 e a especificação de persistência ausente ainda impedem o gate.

## Itens já resolvidos

Não são mais pendências arquiteturais:

- [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md): SQLite e recovery do estado operacional estão decididos; permanece pendente a especificação técnica do modelo persistente.
- [ADR-0010 — Política de encerramento de jobs](adr/runtime/encerramento-de-jobs.md): aviso aos clientes, espera por respostas e timeout estão `refined`.
- [CTR-0002 — Execução de script](especificacao/contratos/execucao-script.md): invocação, working directory, ambiente permitido, concorrência, limites de saída e cancelamento estão `refined`.
- Telegram: confirmação de update, avanço de `offset`, confirmação de entrega, retry de envio, recovery de respostas e retry de leitura por `getUpdates` estão definidos. O módulo e o fluxo permanecem em `refinement` enquanto dependências como CTR-0001 não estiverem refinadas.
- CTR-0001: argumentos, identidade, correlação, envelope de resultado/erro e referências de mídia `{name, path}` foram definidos. As pendências de lifecycle, tipo e limites ficam centralizadas em CTR-0006.
- ADR-0011: arquitetura de processadores registrados e eventos `loading | content | finally` está `refined`.
- ADR-0012: `/tmp/sabia/media/<request_id>/`, múltiplos `content` e streaming foram definidos; lifecycle e recovery permanecem em CTR-0006.

## Condição para liberar o MVP

Antes de iniciar implementação do escopo dependente:

1. documentos ainda em `refinement` precisam atingir `refined` com conteúdo suficiente;
2. especificações ausentes necessárias à implementação precisam ser criadas e refinadas;
3. documentos marcados como `refined` mas incompletos precisam ser corrigidos;
4. inconsistências entre ADRs, desenho e especificação precisam ser reconciliadas;
5. nenhuma lacuna pode ser preenchida por suposição.
