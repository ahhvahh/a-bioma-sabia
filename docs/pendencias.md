# Pendências documentais

Este documento consolida as lacunas ainda abertas na documentação do Sabiá. O detalhe normativo permanece nos documentos referenciados.

## Estado do gate

O gate completo de especificação do MVP permanece `BLOCKED`.

Além de documentos ainda em `refinement`, existem documentos marcados como `refined` cujo conteúdo ainda não é suficiente para implementação sem decisões implícitas. Esses casos precisam ser corrigidos antes de considerar o gate satisfeito.

## BLOCKED

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
  - no MVP, processadores assíncronos são somente scripts Bash previamente cadastrados;
  - cada processador referencia um `script_id` existente no Script Registry;
  - o Sabiá escreve `request_id` UUID v4 no stdin como uma linha; argumentos da operação seguem via argv;
  - stdout é reservado a JSON Lines UTF-8 com eventos `loading | finally`, consumidos incrementalmente linha a linha; stderr permanece diagnóstico;
  - aplicações dedicadas e serviços por socket ficam fora do MVP e serão tratados como evolução futura;
  - `content_type` é metadado declarado: `null` é válido, vazio vira `null`, valor informado precisa ser media type válido e é normalizado para minúsculas; não há inferência por extensão ou inspeção dos bytes;
  - no Telegram, `image/*` usa envio de imagem, `video/*` usa vídeo e demais tipos/`null` usam documento/arquivo genérico;
  - processadores usam timeout absoluto de 2 horas no MVP; `loading` não renova o prazo; ausência de `finally` até o limite encerra o job em `timeout`;
  - mídia não trafega no canal de controle;
  - mídia entra por Unix domain socket local;
  - envelope de mídia usa MessagePack;
  - o canal simples aceita arquivo integral de até `20000000` bytes;
  - o segundo Unix socket aceita mídia de qualquer tamanho permitido pelo contrato em chunks de até `5000000` bytes;
  - para arquivos de até 20 MB, o produtor pode escolher canal simples ou fracionado;
  - acima de 20 MB, o canal fracionado é obrigatório;
  - o arquivo lógico fracionado é limitado a `100000000` bytes (100 MB);
  - abertura do arquivo fracionado envia metadados e recebe `media_id` gerado pelo banco;
  - depois da abertura, cada chunk contém apenas `media_id`, `sequence_id` e BLOB;
  - a primeira sequência esperada é `1`;
  - chunks são persistidos imediatamente e `next_sequence_id` só avança após commit;
  - os metadados fracionados incluem `total_bytes`;
  - `received_bytes` é atualizado a cada chunk;
  - `completed` ocorre automaticamente quando `received_bytes == total_bytes`;
  - chunk que ultrapassaria `total_bytes` é recusado;
  - `streamed_bytes` contabiliza os bytes consumidos na tentativa de envio;
  - payload/chunks permanecem preservados até confirmação remota para permitir retry;
  - sequência pulada retorna `sequence_gap` com `expected_sequence_id`;
  - sequência já recebida retorna `sequence_already_received` sem nova persistência;
  - o Telegram recebe um único arquivo lógico; chunks não são mensagens independentes;
  - CTR-0007 voltou a `refined` após fechamento da completude, sequência e limite total da mídia fracionada;
  - uma requisição pode enviar vários arquivos por vários uploads;
  - uploads repetidos, inclusive com mesmo nome/conteúdo, são aceitos como mídias independentes e não são deduplicados;
  - o serviço apenas valida a existência do `request_id` antes de persistir;
  - o BLOB e sua transmissão são persistidos antes do ACK;
  - ACK retorna `media_id`;
  - limpeza de payload entregue ocorre somente sem transmissões `pending` ou `transmitting`;
  - limpeza é elegível quando a fila fica ociosa ou após 1 hora desde a última limpeza, mas é adiada se houver transmissão ativa;
  - a limpeza remove BLOB/chunks entregues e preserva metadados de rastreabilidade;
  - transmissões `pending | transmitting` são recuperáveis após restart;
  - nenhuma dependência de `/tmp` permanece no contrato de mídia;
  - acesso local aos sockets usa owner `sabia`, group `abioma` e mode `0660`; membros de `abioma` podem usar as interfaces locais e o link configurado da aplicação, sem acesso direto implícito ao PostgreSQL ou aos diretórios internos.
- Informação ausente:
  - caminhos finais dos sockets simples e fracionado;
- Dependências afetadas:
  - [MOD-0004 — Jobs](especificacao/modulos/jobs.md)
  - [FLW-0002 — Job assíncrono](especificacao/fluxos/job-assincrono.md)
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)

### Persistência operacional em PostgreSQL — proposta em revisão

- Decisão relacionada: [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md)
- Documentos criados:
  - [PST-0001 — Proposta de tabelas](especificacao/persistencia/tabelas-estado-operacional.md) — `refinement`
  - [PST-0002 — Relacionamentos](especificacao/persistencia/relacionamentos-estado-operacional.md) — `refinement`
  - [PST-0100 — Entidades persistentes](especificacao/persistencia/entidades/README.md) — `refinement`
  - [CTR-0010 — Origem e conversão dos dados persistentes](especificacao/contratos/origem-dados-persistencia.md) — `refinement`
  - [FLW-0009 — Registro do estado operacional](especificacao/fluxos/registro-estado-operacional.md) — `refinement`
  - [FLW-0010 — Consumo do estado operacional](especificacao/fluxos/consumo-estado-operacional.md) — `refinement`
- Estado atual: cada entidade proposta possui documento próprio com campos, tipo PostgreSQL, origem, conversão e exemplo ilustrativo. O contrato CTR-0010 consolida origem → persistência e os fluxos FLW-0009/FLW-0010 separam registro e consumo.
- Estado necessário: documentos de persistência necessários ao MVP em `refined`.
- Pontos ainda `BLOCKED`:
  - caminhos brutos exatos dos campos Telegram usados pelo adaptador;
  - direção física final da relação `inbound_update/request`;
  - política final de FKs e `ON DELETE`;
  - consistência entre `media.request_id` e `media_transmission.request_id`;
  - formato persistente final de `outbound_message.content`;
  - momento normativo de atualização de `alert_state.last_notified_at`;
  - retenção de `alert_state` após remoção de schedule;
  - mecanismo concreto de migração/versionamento;
  - política de retenção/purge dos registros operacionais concluídos depois que entram no processo de limpeza;
  - modelo de `job_attempt` somente se retry automático for habilitado.
- Dependências afetadas:
  - [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)
  - [MOD-0004 — Jobs](especificacao/modulos/jobs.md)
  - [MOD-0005 — Scheduler e Alert Manager](especificacao/modulos/scheduler-alertas.md)
  - [CTR-0003 — Job](especificacao/contratos/job.md)
  - [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md)
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

### ADR-0007 — referência de persistência de alertas a reconciliar

[ADR-0007 — Scheduler e alertas orientados a estado](adr/monitoramento/scheduler-alertas-estado.md) ainda afirma que a persistência do estado entre reinícios não está decidida. ADR-0009 já definiu PostgreSQL e recovery desse estado; qualquer referência anterior ao mecanismo de persistência deve apontar para a decisão vigente.

### FLW-0004 — estado em refinement sem lacuna própria explícita

[FLW-0004 — Encerramento do serviço](especificacao/fluxos/encerramento-servico.md) está em `refinement`, embora o fluxo já detalhe gatilho, sequência, timeout, falhas e recovery. Antes de mudar o status, deve ser verificado se a dependência em MOD-0005 e a especificação de persistência ausente ainda impedem o gate.

## Itens já resolvidos

Não são mais pendências arquiteturais:

- [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md): PostgreSQL, banco lógico/role próprios e recovery do estado operacional estão decididos; permanece pendente a especificação técnica do modelo persistente e do contrato concreto de conexão.
- [ADR-0010 — Política de encerramento de jobs](adr/runtime/encerramento-de-jobs.md): aviso aos clientes, espera por respostas e timeout estão `refined`.
- [CTR-0002 — Execução de script](especificacao/contratos/execucao-script.md): invocação, working directory, ambiente permitido, concorrência, limites de saída e cancelamento estão `refined`.
- [PST-0003 — Claim concorrente de filas persistentes](especificacao/persistencia/claim-concorrente-filas.md): jobs, mensagens e transmissões usam claim atômico no PostgreSQL com `FOR UPDATE SKIP LOCKED`, transição de estado e commit antes do processamento externo; mensagens usam o estado intermediário `sending`.
- `request.status`: definido como `received → processing → completed | failed`, representando apenas o processamento pelo Core; jobs e entregas mantêm estados independentes.
- `request_id`: UUID v4 gerado pelo Sabiá, persistido como PostgreSQL `uuid` e propagado como string canônica em JSON/MessagePack; ao término da tarefa, a correlação torna-se elegível para limpeza, respeitando dependências ainda não terminais.
- Jobs: uma `request` pode criar `0..N` jobs; `job.request_id` não é `UNIQUE`; no MVP, jobs simultaneamente em `running` são limitados à quantidade de threads lógicas de CPU disponíveis ao processo no startup, enquanto `jobs.max_pending` limita apenas a fila `queued`.
- Telegram: confirmação de update, avanço de `offset`, confirmação de entrega, retry de envio, recovery de respostas, retry de leitura por `getUpdates` e seleção de apresentação de mídia por `content_type` estão definidos. O módulo e o fluxo permanecem em `refinement` por outras dependências ainda abertas.
- [CTR-0001 — Comando interno](especificacao/contratos/comando-interno.md): `refined`; argumentos, identidade, correlação, envelope de resultado/erro e regra normativa de `content_type` estão definidos.
- [CTR-0006 — Mídia persistida](especificacao/contratos/midia-persistida.md): `refined`; `content_type` é opcional, normalizado e não é inferido pelo nome ou pelos bytes; Telegram usa o tipo apenas para escolher a apresentação da mídia.
- ADR-0011/CTR-0005: no MVP, processadores assíncronos são scripts Bash registrados; recebem `request_id` por stdin, argumentos via argv e devolvem `loading | finally` como JSON Lines em stdout; mídia permanece separada no CTR-0008.
- ADR-0012/ADR-0013 + CTR-0006/CTR-0007/CTR-0008/CTR-0009: mídia não depende de `/tmp`; canal simples é limitado a 20 MB e o canal fracionado aceita arquivos de até 100 MB, sendo obrigatório acima de 20 MB. O segundo socket abre um `media_id` persistente, recebe chunks de até 5 MB com sequência estrita e determina completude por `total_bytes`. CTR-0007 voltou a `refined`; FLW-0008 define limpeza segura somente com fila de transmissão ociosa.

## Condição para liberar o MVP

Antes de iniciar implementação do escopo dependente:

1. documentos ainda em `refinement` precisam atingir `refined` com conteúdo suficiente;
2. especificações ausentes necessárias à implementação precisam ser criadas e refinadas;
3. documentos marcados como `refined` mas incompletos precisam ser corrigidos;
4. inconsistências entre ADRs, desenho e especificação precisam ser reconciliadas;
5. nenhuma lacuna pode ser preenchida por suposição.
