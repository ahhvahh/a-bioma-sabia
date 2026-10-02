# Pendências documentais

Este documento consolida as lacunas ainda abertas na documentação do Sabiá. O detalhe normativo permanece nos documentos referenciados.

## Estado do gate

O gate completo de especificação do MVP permanece `BLOCKED`.

Além de documentos ainda em `refinement`, existem documentos marcados como `refined` cujo conteúdo ainda não é suficiente para implementação sem decisões implícitas. Esses casos precisam ser corrigidos antes de considerar o gate satisfeito.

## BLOCKED

### CTR-0001 — Comando interno

- Documento: [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md)
- Estado atual: `refinement`.
- Estado necessário: `refined`.
- Estado atual do conteúdo:
  - argumentos usam `string[]`;
  - identidade e correlação usam `request_id`, `client_id`, `principal_id`, `reply_context` e `received_at`;
  - saída usa `message | job | file | error`;
  - mídia de entrada e saída é referenciada por `media_id`;
  - o Core não recebe BLOB nem caminho de filesystem.
- Informação ausente:
  - regra normativa para `content_type`;
  - política de retenção do BLOB após entrega.
- Dependências afetadas:
  - [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md)
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)
  - [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md)

### Processadores assíncronos e ingestão de mídia

- Decisões relacionadas:
  - [ADR-0011 — Processadores assíncronos registrados](adr/processamento/processadores-assincronos-registrados.md)
  - [ADR-0012 — Ingestão persistente de mídia por Unix socket](adr/processamento/ingestao-midia-socket-messagepack.md)
  - [ADR-0013 — Dois canais de ingestão e upload fracionado](adr/processamento/ingestao-midia-fracionada.md)
- Documentos:
  - [MOD-0007 — Processadores assíncronos e transporte](especificacao/modulos/processadores-assincronos.md)
  - [MOD-0008 — Ingestão e armazenamento de mídia](especificacao/modulos/ingestao-midia.md)
  - [CTR-0005 — Protocolo de processador assíncrono](especificacao/contratos/processador-assincrono.md)
  - [CTR-0006 — Mídia persistida](especificacao/contratos/midia-persistida.md)
  - [CTR-0007 — Transmissão persistente de mídia](especificacao/contratos/transmissao-midia.md)
  - [CTR-0008 — Ingestão de mídia por Unix socket e MessagePack](especificacao/contratos/ingestao-midia-messagepack.md)
  - [CTR-0009 — Ingestão fracionada de mídia por Unix socket](especificacao/contratos/ingestao-midia-fracionada.md)
  - [FLW-0005 — Processamento assíncrono](especificacao/fluxos/processamento-assincrono.md)
  - [FLW-0006 — Ingestão local de mídia](especificacao/fluxos/ingestao-midia.md)
  - [FLW-0007 — Ingestão fracionada de mídia](especificacao/fluxos/ingestao-midia-fracionada.md)
  - [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md)
- Estado atual: decisões arquiteturais `refined`; especificações ainda em `refinement` onde indicado.
- Decisões já fechadas:
  - canal de controle do processador usa `loading | finally`;
  - mídia não trafega no canal de controle;
  - mídia entra por Unix domain socket local;
  - envelope de mídia usa MessagePack;
  - o canal simples aceita arquivo integral de até `20000000` bytes;
  - arquivos maiores usam um segundo Unix socket com chunks de até `5000000` bytes;
  - abertura do arquivo fracionado envia metadados e recebe `media_id` gerado pelo banco;
  - depois da abertura, cada chunk contém apenas `media_id`, `sequence_id` e BLOB;
  - a primeira sequência esperada é `1`;
  - chunks são persistidos imediatamente e `next_sequence_id` só avança após commit;
  - sequência pulada retorna `sequence_gap` com `expected_sequence_id`;
  - sequência já recebida retorna `sequence_already_received` sem nova persistência;
  - o Telegram recebe um único arquivo lógico; chunks não são mensagens independentes;
  - CTR-0007 voltou para `refinement` porque a entrega fracionada depende da completude definida por CTR-0009;
  - uma requisição pode enviar vários arquivos por vários uploads;
  - uploads repetidos, inclusive com mesmo nome/conteúdo, são aceitos como mídias independentes e não são deduplicados;
  - o serviço apenas valida a existência do `request_id` antes de persistir;
  - o BLOB e sua transmissão são persistidos antes do ACK;
  - ACK retorna `media_id`;
  - transmissões `pending | transmitting` são recuperáveis após restart;
  - nenhuma dependência de `/tmp` permanece no contrato de mídia.
- Informação ausente:
  - framing/protocolo concreto do canal de controle CTR-0005;
  - caminhos finais dos sockets simples e fracionado;
  - ownership, grupo e modo de acesso dos sockets;
  - indicador de completude/último chunk;
  - limite total do arquivo lógico fracionado;
  - política de retenção do conteúdo após entrega;
  - regra para determinar/validar `content_type`;
  - schema final da configuração `processors`.
- Dependências afetadas:
  - [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md)
  - [MOD-0004 — Jobs](especificacao/modulos/jobs.md)
  - [FLW-0002 — Job assíncrono](especificacao/fluxos/job-assincrono.md)
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)

### Persistência operacional — especificação técnica ausente

- Decisão relacionada: [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md)
- Estado atual: decisão arquitetural `refined`; CTR-0007 já define de forma implementável a persistência e o recovery das transmissões de mídia, mas o restante do modelo operacional ainda não possui especificação técnica completa.
- Estado necessário: especificação de persistência `refined`.
- Informação ausente:
  - entidades/tabelas e relações para requisições, jobs, tentativas, respostas textuais e estado de alertas;
  - campos, tipos, chaves e restrições dessas entidades;
  - regras de atomicidade e idempotência ainda não cobertas por CTR-0007;
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
- CTR-0001: argumentos, identidade, correlação e envelope de resultado/erro foram definidos; mídia agora é referenciada por `media_id`.
- ADR-0011: processadores registrados usam canal de controle `loading | finally`; mídia foi separada para o socket CTR-0008.
- ADR-0012/ADR-0013 + CTR-0006/CTR-0007/CTR-0008/CTR-0009: mídia não depende de `/tmp`; canal simples é limitado a 20 MB e o segundo socket abre um `media_id` persistente e recebe chunks de até 5 MB com sequência estrita. O MVP não deduplica uploads integrais.

## Condição para liberar o MVP

Antes de iniciar implementação do escopo dependente:

1. documentos ainda em `refinement` precisam atingir `refined` com conteúdo suficiente;
2. especificações ausentes necessárias à implementação precisam ser criadas e refinadas;
3. documentos marcados como `refined` mas incompletos precisam ser corrigidos;
4. inconsistências entre ADRs, desenho e especificação precisam ser reconciliadas;
5. nenhuma lacuna pode ser preenchida por suposição.
