<!-- A-BIOMA-DOC-LEGADO -->

# A-Bioma Sabiá — Regras consolidadas

> **Absorção parcial:** [REF-005 — Mapa de rastreabilidade](referencia/REF-005-rastreabilidade-regras.md). Os 14 assuntos foram mapeados, mas contratos administrativos e de assinaturas por eventos permanecem incompletos. **Esta origem ainda não recebeu o marcador MIGRADO.**

![Tipo](https://img.shields.io/badge/Documento-Regras_consolidadas-555555?style=flat-square)
![Status](https://img.shields.io/badge/Status-s%C3%ADntese_em_revis%C3%A3o-d4a72c?style=flat-square)
![Versão](https://img.shields.io/badge/Vers%C3%A3o-0.1-6e7781?style=flat-square)
![Escopo](https://img.shields.io/badge/Escopo-MVP_e_evolu%C3%A7%C3%A3o-0969da?style=flat-square)

> **Fonte:** consolidação de `docsLegado.md` (279 documentos e artefatos originais). Este texto reúne regras sem repetir Charter, Scope, ADRs e contratos. Os identificadores e caminhos citados referem-se aos documentos **históricos dentro do legado**, não a arquivos que precisem continuar existindo separadamente. **Não é evidência de implementação:** o material consultado informa que o serviço, o `sabiactl` e os sockets ainda não estão implementados. Itens sem decisão final aparecem em **Pendências**, sem assumir soluções inexistentes.

## 1. Propósito e limites

O **Sabiá** é um serviço Linux do ecossistema A-Bioma que conecta usuários autorizados do Telegram, operações locais previamente registradas, trabalhos assíncronos, monitoramento, alertas, assinaturas de eventos e transferência de mídia. O **Core é independente da Telegram Bot API**: o adaptador traduz mensagens e identidades para comandos internos e reconverte os resultados ao transporte correto.

**Regras estruturais:**

1. Executar em **Debian 13 (Trixie)**, como serviço `systemd`, sem privilégios de `root`, sob usuário dedicado `sabia`. A implementação principal prevista é em **Go**; binário e interface administrativa permanecem pendentes.
2. Manter múltiplos **clientes/bots Telegram independentes**, inicialmente `bioma`, `tools` e `alerts`; cada um possui credencial, usuários, chats, catálogo de comandos e permissões próprios. Nunca compartilhar autorização implicitamente.
3. Receber updates via **long polling HTTPS (`getUpdates`)**, sem necessidade de webhook público. Interfaces locais usam **Unix domain sockets (UDS)**; não expor HTTP/REST, gRPC, TCP ou shell remoto como interfaces do primeiro MVP.
4. Aceitar somente **operações cadastradas**; mensagens Telegram podem fornecer parâmetros validados, **nunca** caminhos de executáveis, interpretadores ou comandos de shell arbitrários.
5. Persistir estado operacional em **PostgreSQL**, com banco lógico e role próprios e sem fallback normativo para SQLite. A primeira versão de PostgreSQL escolhida no Discovery é a **17**; a migração para 18 será avaliada posteriormente.
6. Preservar a correlação entre **cliente, usuário, chat, requisição, execução e resposta**, incluindo falhas e reinicializações.

**Fora do primeiro MVP:** processadores complexos de imagem, vídeo e áudio; serviços/processadores externos via sockets; canais remotos extras; comandos que executem shell arbitrário. A infraestrutura de mídia e jobs **faz parte** do MVP mesmo que os processadores complexos não façam.

## 2. Identidades, isolamento e autorização

| Identidade / grupo | Regra de acesso prevista |
|---|---|
| `sabia` | Usuário dedicado do daemon, sem login interativo e sem execução como `root`; administra os dados internos. |
| `absabia` | Grupo de operadores e aplicações locais autorizado a **conectar aos sockets do Sabiá**, mas não a acessar o PostgreSQL ou os diretórios internos diretamente. |
| `abioma_db` | Grupo reservado ao UDS do PostgreSQL; acesso restrito ao serviço e à administração do banco estritamente necessária. |
| Usuário Telegram | Autorização por **bot + Telegram User ID** e, quando configurado, por chat; desconhecer/recusar uma ação deve impedir sua execução. |

- O **`sabiactl` comunica-se apenas com o serviço por UDS**. O serviço realiza validação, operações e persistência; o terminal não recebe credenciais de banco nem escreve diretamente nas pastas internas.
- Conectar a um socket por pertencer a `absabia` **não concede direitos administrativos irrestritos**. A identidade local e a permissão da operação devem ser verificadas pelo Sabiá; operações sensíveis exigem auditoria.
- Sockets de aplicação: dono/grupo **pretendidos** `sabia:absabia`, modo `0660`, em diretórios também protegidos. PostgreSQL: conexão do Sabiá **somente por UDS**, com proteção por grupo, `pg_hba.conf`, role SQL de privilégio mínimo e bloqueio de rotas TCP não autorizadas.
- Segredos e tokens não devem constar de documentação, repositório, logs ou respostas; os processos filhos não herdam variáveis de ambiente além das expressamente permitidas.
- **Divergência ainda aberta:** contratos de mídia anteriores exigem grupo `abioma`, enquanto o modelo de acesso local mais recente prevê `absabia`. A decisão deve ser harmonizada antes de implementar.

## 3. Bots, contatos e solicitações de acesso

### 3.1 Configuração dos bots

- Cada bot é criado no Telegram pelo **@BotFather**, obtendo **token exclusivo**; a criação do bot não instala nem inicializa o Sabiá.
- A configuração planejada é `telegram.clients`, com `enabled`, `token_env`, `allowed_users`, `allowed_chats` opcional e operações permitidas por cliente.
- A variável apontada por `token_env` fornece a credencial; o token real não deve ser gravado no YAML versionável.
- Cada cliente ativo possui seu próprio polling, offset, backoff e destino de resposta. Desativar ou falhar um cliente não deve suspender os demais.

### 3.2 Cadastro e aprovação de usuários

1. A pessoa abre o bot e envia `/start`.
2. O Sabiá identifica **cliente, Telegram User ID e chat**, registra/atualiza o contato e, quando não autorizado, gera uma **solicitação interna de acesso pendente**. Identificação não equivale a autorização.
3. No `sabiactl`, o operador escolhe **Usuários e clientes → Solicitações de acesso → Bot** e percorre a fila de solicitações desse cliente.
4. Na lista: **`↑`/`↓`** selecionam a pessoa; **`→`** prepara aprovação; **`←`** prepara rejeição/remoção da fila.
5. **`Y`** confirma qualquer decisão; **`N` ou `Esc`** cancela sem alterar o estado. O diálogo deve mostrar bot, nome e **ID numérico do Telegram**; username/nome são apenas informativos.
6. A decisão deve ser validada novamente no serviço, persistida de forma transacional e auditada. Rejeitar retira da fila, **sem apagar o histórico**; aprovar libera apenas o bot autorizado.
7. Ao usar `/start` ou `/menu` novamente, a pessoa recebe **somente as ações permitidas** ao seu contexto.

Esta fila é **interna ao Sabiá**, não uma solicitação nativa de entrada em grupo Telegram e não uma assinatura paga. O fluxo administrativo e as tabelas de autorização são **requisitos propostos**, ainda não implementados. A configuração antiga por `allowed_users` no YAML exige restart; aprovações dinâmicas no PostgreSQL precisam ser conciliadas com essa regra.

## 4. Cadastro de arquivos, scripts e ações

A ordem funcional de administração é: **arquivo → validação → incorporação do script → ação de um bot → publicação**. Se um script já foi validado, ele pode ser reutilizado por outro vínculo **explícito**, sem novo upload.

### 4.1 Arquivo e script

1. No `sabiactl`, escolher **Scripts → Enviar novo script**, indicar arquivo **local** e metadados (identificador lógico, descrição, tipo, modo e ambiente permitido).
2. Realizar **pré-validação local**: existência, legibilidade, tamanho, arquivo regular e metadados básicos.
3. Transmitir o conteúdo por **API UDS administrativa**, para uma **área de validação isolada** no Sabiá. Enviar o arquivo **não significa aceitá-lo ou instalá-lo**.
4. No serviço, verificar integridade, tipo permitido, sintaxe (para Bash, por exemplo, `bash -n`), interpretador e ambiente admitidos, metadados e conformidade com o protocolo de execução.
5. Quando testes com execução forem exigidos, usar ambiente **isolado, sem segredos e privilégios do daemon**; sintaxe válida, isoladamente, **não comprova segurança**. Sem isolamento adequado, não ativar automaticamente: manter em revisão.
6. Exibir relatório humano de validação. Somente após **sucesso e confirmação `Y`** o serviço incorporará o script ao catálogo interno. Um arquivo rejeitado permanece indisponível para execução ou vinculação a ações.

**Regras do executor:** somente `script_id` previamente registrado; caminho de executável/interpretador/diretório de trabalho **absoluto** e controlado pelo operador; argumentos remotos previamente validados em `argv`; execução **sem `sh -c` ou `bash -c` automático**; lista explícita de variáveis de ambiente herdáveis; timeout e cancelamento por execução; processos concorrentes isolados entre si. Para execução convencional, a captura de `stdout` e `stderr` é limitada a **1 MiB por canal** no MVP.

### 4.2 Ação apresentada no Telegram

1. No `sabiactl`, abrir **Menus e ações → Selecionar bot → Nova ação**.
2. Informar **identificador, nome visível, posição/caminho do menu, `script_id` validado, modo de execução, parâmetros aceitos e permissões**.
3. Validar bot, identificação da ação, vínculo, argumentos e autorização. A ação deve nascer **desabilitada/rascunho** até publicação.
4. Revisar e pressionar **`Y`** para solicitar publicação; o serviço revalida e persiste o catálogo, sem acesso SQL direto do terminal.
5. O `/start` ou `/menu` do bot passa a mostrar a ação **somente** para usuários/chats autorizados. O clique aciona o Command Router e a operação previamente cadastrada.

A hierarquia pode ser derivada do **caminho lógico da ação**, sem exigir uma tabela independente de árvore. O formato da API UDS administrativa, o armazenamento de scripts enviados e o esquema definitivo do catálogo **não estão fechados**.

## 5. Comando interno e navegação

O Core recebe dados neutros em relação ao transporte:

| Campo | Significado |
|---|---|
| `request_id` | **UUID v4** gerado pelo Sabiá, estável durante processamento e entrega. |
| `client_id`, `principal_id` | Bot lógico e identidade autorizada do solicitante. |
| `command`, `arguments[]` | Operação lógica e lista ordenada de strings; validação semântica pertence à operação. |
| `reply_context` | `transport` e `destination_id` opaco para o Core; destino de resposta original. |
| `received_at`, `attachments[]` | Instante de aceite e referências opcionais a arquivos persistidos por `media_id`. |

O retorno do Core admite **`type = message | job | file | error`** e preserva `request_id`. Erros controlados incluem **`code`, `message`, `retryable`**; resultados não expõem objetos internos da Bot API.

No Telegram, `/start` e `/menu` mostram a árvore de ações por `InlineKeyboardMarkup`. Navegação usa `callback_query`, `answerCallbackQuery`, `editMessageText`, **Voltar** e **Início**. `callback_data` tem até **64 bytes** e deve transportar identificador opaco, **nunca** caminho de script, segredo ou autorização implícita. A cada clique devem ser revalidados cliente, usuário, chat e operação.

### Comandos do primeiro MVP

| Bot | Comandos previstos |
|---|---|
| `bioma` | `/start`, `/help`, `/status`, `/cpu`, `/memory`, `/disk`, `/uptime`, `/run` |
| `tools` | `/help`, `/jobs`, `/job` |
| `alerts` | `/status` |

Comandos como processamento complexo de imagem/áudio/vídeo, `/convert`, `/resize`, `/compress` e `/cancel` permanecem fora das primeiras entregas. A **sintaxe, argumentos e respostas de parte dos comandos do MVP** ainda não estão detalhados o suficiente para implementação.

## 6. Telegram: recebimento, entrega e recuperação

- Para cada bot, receber updates por `getUpdates`, **persistir `raw_update` antes de confirmar avanço do offset** e aplicar unicidade por `(client_id, transport, update_id)`.
- Após commit transacional de update aceito, consultar com **`offset = maior update_id persistido + 1`**. Update repetido não pode criar segunda execução lógica.
- Manter **inbox transacional**, processamento recuperável e **outbox persistente** para respostas e mídias. A confirmação de processamento **não equivale** à confirmação de entrega.
- Polling de cada cliente possui backoff temporário de **1, 2, 4, 8, 16 até 30 segundos**; `429` respeita `retry_after`; `401/403` suspendem somente o cliente afetado; polling bem-sucedido reinicia o backoff.
- Reservar uma mensagem pendente antes de enviar: **`pending → sending → delivered | pending | failed`**. Registrar o **`message_id` remoto** antes de considerar `delivered`. Após reinício, `sending` volta a `pending`.
- Envio usa semântica **`at-least-once`**: se a Bot API aceitar a mensagem, mas a confirmação local se perder, uma nova tentativa pode resultar em **duplicação de entrega**, sem executar novamente o comando.
- Todas as respostas e mídias retornam pelo **mesmo bot e destino** correlacionados à requisição; falha de um cliente não pode alterar outro.
- O adaptador Telegram é o único responsável por chamadas à Bot API. O Core não conhece tokens, SDK, chat objects ou caminhos de arquivo para transporte.

## 7. Jobs e processadores

| Item | Regra |
|---|---|
| Identidade | `job_id` inteiro sequencial PostgreSQL; cada `request_id` pode criar **zero ou vários jobs**. |
| Estados | `queued → running → completed / failed / cancelled / timeout`; `queued → cancelled` também permitido. Estados terminais não reabrem. |
| Capacidade | Jobs `running` limitados ao número de **threads lógicas de CPU disponíveis no startup**; `jobs.max_pending` positivo limita a fila. Não há `jobs.max_workers` configurável no MVP. |
| Retry | **Desabilitado por padrão** (`max_retries = 0`); repetição automática só para operação explicitamente repetível e com contrato de tentativa definido. |
| Reinício | `queued` continua elegível; `running` interrompido vira **`failed/service_restart`**, sem reexecução automática; mensagens prontas continuam pendentes. |
| Consulta | `/jobs` e `/job <id>` retornam somente jobs visíveis ao cliente e solicitante autorizados. |

**Processador assíncrono no MVP:** somente **script Bash cadastrado** que recebe `request_id` UUID v4 em uma linha de **`stdin`**, argumentos em `argv` e emite **JSON Lines UTF-8** exclusivamente no `stdout`, com `{ "request_id", "status": "loading" | "finally", "message" }`. `stderr` é diagnóstico. `loading` atualiza progresso; **`finally`** encerra semanticamente o processamento. O timeout é **2 horas absolutas**, sem extensão por mensagens de progresso. Processo terminado sem `finally` não implica sucesso; expiração gera `processor_timeout`; falha de transporte encerra a execução.

**Shutdown:** ao receber `SIGTERM`/`SIGINT`, parar novos trabalhos, suspender scheduler, tentar avisar todos os clientes ativos e aguardar jobs `running` até `service.shutdown_timeout`. Os não concluídos são encerrados como **`failed/shutdown_timeout`**; `queued` e respostas não entregues permanecem persistidos para recuperação.

## 8. Mídia: contratos e limites

A mídia binária é persistida no PostgreSQL, referenciada por **`media_id`**, vinculada ao `request_id` e enviada **separadamente** dos eventos de controle do processador. O nome lógico do arquivo **não é caminho do filesystem**; nome e `content_type` não autorizam operações.

| Canal | Transporte e entrada | Limites do MVP | Confirmação |
|---|---|---|---|
| Ingestão integral — **CTR-0008** | UDS próprio; **um** objeto MessagePack com `version=1`, `request_id`, `name`, `content_type?`, `data` binário por conexão | `data` ≤ **20.000.000 bytes** | ACK `accepted` com `media_id` somente **depois do commit** de mídia e transmissão. |
| Ingestão fracionada — **CTR-0009** | Outro UDS; abertura com `request_id`, `name`, `content_type?`, `total_bytes`; depois chunks com `media_id`, `sequence_id`, `data` | Total ≤ **50.000.000 bytes**; cada chunk ≤ **5.000.000 bytes**; sequência começa em **1** | ACK por chunk somente após commit; arquivo completo quando `received_bytes == total_bytes`. |

- Acima de 20 MB, o canal fracionado é **obrigatório**; abaixo disso, ambos são válidos. Partes fora de ordem ou já recebidas são rejeitadas com códigos controlados; arquivo incompleto não pode ser entregue.
- Cada upload integral confirmado gera um `media_id` novo, mesmo com conteúdo/nome repetidos; **não há idempotência de upload** no MVP. Chunks são deduplicados pela posição (`media_id`, `sequence_id`), sem hash comparativo.
- Os dados e metadados não podem ser devolvidos em ACK antes da transação persistir. O produtor não fornece cliente ou destino de resposta: ambos são recuperados pela correlação de `request_id`.
- Conteúdo binário não aparece em logs nem no protocolo de progresso. Um mesmo `request_id` pode produzir **várias mídias independentes**.
- `content_type`: `null` é permitido; valor não vazio deve ser media type sintaticamente válido em minúsculas; extensão e bytes não são usados para inferi-lo **no MVP**. Para envio: JPEG/PNG compatíveis por `sendPhoto`, MP4 compatível por `sendVideo`, demais por `sendDocument`, sempre conforme limites aplicáveis da Bot API.
- Para transmissão ao Telegram, persistir **`pending → transmitting → delivered | pending | failed`**; na retomada, reprocessar transmissões pendentes/em andamento elegíveis. Uma mídia fracionada é reconstruída em **stream sequencial**, sem exigir arquivo completo em RAM.
- Os limites do **Telegram público são diferentes** dos limites internos: a documentação registra até 20 MB para download comum por `getFile` e até 50 MB para alguns envios, sujeitos às regras específicas do método. Não presumir que todo arquivo interno de 50 MB poderá ser baixado/enviado por todos os métodos.

## 9. Scheduler, alertas e assinaturas de eventos

**Scheduler (MVP):** tarefas previamente cadastradas no YAML, executadas por **`interval` positivo**, sem cron nem calendário absoluto. Disparo que encontra a **mesma tarefa já em execução** é ignorado e registrado; não abre execução sobreposta. O monitoramento interpreta saída `0 = OK`, `1 = WARNING`, `2 = CRITICAL`, `3 = UNKNOWN`; falhas técnicas e timeout tornam o resultado `UNKNOWN`.

**Alertas orientados a estado:** comparar com a última avaliação **persistida**. Primeira avaliação `OK` é silenciosa; a primeira degradada alerta. Mudanças de estado e recuperação para `OK` alertam; estados repetidos não geram mensagens, exceto **lembrete configurado** (`reminder_interval`) para `WARNING`, `CRITICAL` ou `UNKNOWN`. Recuperação encerra lembretes. Política de relatório “sempre enviar” é prevista, mas seu esquema definitivo está pendente.

**Assinaturas por evento (objetivo ativo OBJ-009):** usuário autorizado poderá **assinar/cancelar tópicos** e receber publicações produzidas por aplicações locais autorizadas. Autorização, sequência por tópico/assinante, histórico e entregas precisam sobreviver a restart; aceitar uma publicação não comprova leitura pelo destinatário. **Não confundir** com o fluxo de aprovação de acesso ao bot nem com **mailings periódicos** (`IN-013`), que permanecem em backlog. Os contratos de publicação/assinatura por evento **ainda precisam de refinamento**.

## 10. Persistência e consistência

**Fonte oficial:** PostgreSQL com transações curtas. As filas persistentes concorrentes devem realizar **claim atômico** por rotina no banco com **`SELECT ... FOR UPDATE SKIP LOCKED`**, transição de estado e commit **antes** de executar script, chamar Telegram ou transmitir mídia. Não manter lock/transação aberta durante operação externa.

| Entidade proposta | Responsabilidade única |
|---|---|
| `transport_cursor` | Cursor confirmado por bot/transporte. |
| `inbound_update` | Inbox bruta e idempotência de updates recebidos. |
| `request` | Identidade, comando, argumentos, origem e destino normalizados. |
| `job` / `job_state_history` | Execução assíncrona e histórico de transições. |
| `outbound_message` | Mensagens textuais pendentes e confirmação de entrega. |
| `alert_state` | Último resultado e momento de notificação do monitoramento. |
| `media` / `media_chunk` | Metadados, binário (`bytea`) e partes persistidas. |
| `media_transmission` | Fila de envio de mídia e estado da entrega. |
| `schema_version` | Controle de migração **somente se** a ferramenta escolhida não fornecer equivalente. |

**Relações essenciais:** `request` origina zero ou muitos jobs, mensagens e mídias; cada mídia possui zero ou muitos chunks/transmissões. Identificadores lógicos (`client_id`, comandos, destinos) são textuais; `request_id` é UUID v4; IDs numéricos são gerados pelo banco; horários absolutos usam `timestamptz`; binários usam `bytea`.

**Integridade:** unicidade da inbox por bot/transporte/update; restrições de estado; transação única para histórico de job e mudança de estado; transação única para chunk e contadores/completude; unicidade/exclusividade do claim; confirmação de mensagem com ID remoto na mesma transação que marca `delivered`. Referências operacionais usam `ON DELETE RESTRICT`; exclusão controlada deve percorrer dependências **das folhas para a raiz**, sem cascata automática.

**Ainda não estão fechados:** modelo físico de **contatos, solicitações de acesso, autorizações, catálogo de ações e arquivos administrativos**; retenção/purge; migração de schema; FK final inbox/request; consistência entre `media_transmission.request_id` e `media.request_id`; correlação de alertas sem uma request de origem. Estes conceitos **não devem** ser considerados tabelas já aprovadas.

## 11. Configuração e operação local

**Arquivo de configuração previsto:** `/etc/sabia/sabia.yaml`. No contrato atual (CFG-0001), é validado integralmente no **startup** e **não há reload dinâmico**; configurações inválidas ou referências de segredo inexistentes impedem iniciar parcialmente o serviço.

| Seção do YAML | Conteúdo |
|---|---|
| `service` | `shutdown_timeout` obrigatório e positivo. |
| `database` | Conexão/credencial da role PostgreSQL dedicada; esquema concreto pendente. |
| `telegram.clients` | Bots, `enabled`, `token_env`, `allowed_users`, `allowed_chats?` e operações habilitadas. |
| `scripts`, `processors` | Registros lógicos de executáveis, `script_id`, caminhos absolutos, ambiente, timeout; processador Bash via referência a script. |
| `media_ingest`, `media_chunk_ingest` | Caminhos **distintos e absolutos** dos sockets, limites fixos e política de acesso. |
| `jobs`, `schedules`, `logging` | `max_pending`, intervalos de tarefas, política de alertas, nível e destino de logs. |

A operação humana planejada se dá pelo **`sabiactl`**: telas navegáveis, cores e textos legíveis, sem presumir valores reais para métricas indisponíveis; consultas de estado/conexões; clientes e contatos; fila de solicitações; testes; administração de ações, menus e scripts. **Toda mutação privilegiada** requer verificação e confirmação; `Y` confirma aprovações/rejeições e publicações, `N`/`Esc` cancela.

**Atenção:** a administração dinâmica prevista (`sabiactl` + PostgreSQL) **diverge** de partes do CFG-0001 (YAML estático com restart). Essa interface deve ser especificada e o modelo de autoridade entre configuração estática e registros persistentes deve ser decidido antes da implementação.

## 12. Testes e critérios mínimos de aceite

1. **Isolamento:** bot A não acessa tokens, autorizações, catálogo ou respostas do bot B; usuários não autorizados e operadores locais sem permissão não executam mutações.
2. **Entrada:** update repetido não cria duas requisições; perda de energia antes/depois do commit não perde o estado aceito nem avança offset indevidamente.
3. **Fila e jobs:** dois workers não assumem o mesmo item; `queued` retoma, `running` interrompido falha com motivo persistido; limite de concorrência e `max_pending` respeitados.
4. **Script e ação:** arquivo inválido não é incorporado; arquivo validado não é automaticamente publicado; ação somente executa script cadastrado e argumentos autorizados.
5. **Progresso e saída:** JSON Lines estrito, `request_id` preservado, `loading` não conclui, timeout sem `finally` falha, stderr não contamina stdout.
6. **Mídia:** limites, sequência, ACK depois de commit, reconstituição por streaming, bloqueio de transmissão incompleta e retry de entrega testados.
7. **Telegram:** falha temporária usa backoff; `retry_after` respeitado; entrega confirmada persiste o ID remoto; mensagens pendentes sobrevivem a restart.
8. **Monitoramento:** primeiro estado saudável silencioso, transição degradada e recuperação notificadas, estado repetido sem spam e disparos sobrepostos ignorados.
9. **Operação:** serviço inicia sem root, sem webhook público e sem SQL direto pelo `sabiactl`; logs não incluem segredos nem binários; shutdown é previsível.

**Provas funcionais iniciais previstas:** comando de sistema via bot `bioma` e verificação `disk-check` por scheduler com alerta no bot `alerts`. Os documentos consultados **especificam** essas provas, mas não comprovam executáveis ou testes realizados.

## 13. Pendências e conflitos que não podem ser inferidos

| Tema | Definição ainda necessária |
|---|---|
| **Grupo do UDS** | Reconciliar `abioma` (contratos antigos) com `absabia` (diretriz recente) e atualizar os donos, grupos e autorizações normativos. |
| **Configuração dinâmica** | Decidir precedência entre YAML e PostgreSQL para bots, usuários, scripts e ações; permitir aprovações/publicações pelo `sabiactl` sem inventar reload automático. |
| **Administração UDS** | Fechar caminho, framing, serialização, operações, erros, autorização por ação, auditoria e concorrência. MessagePack **só está especificado para mídia**, não para a API administrativa. |
| **Banco** | Fechar conexão PostgreSQL por UDS, autenticação SQL, esquema de autorização/contatos/catálogo, migrações, relacionamentos pendentes, retenção e tratamento de dados incompatíveis. |
| **Telegram** | Definir contrato operacional de `/run`, `/jobs`, `/job`, parâmetros e respostas; de-para dos metadados de fotos/vídeos recebidos. |
| **Scheduler** | Resolver mensagem de alerta sem `request_id`, momento de `last_notified_at`, remoção de agendamentos e formato final da política de relatório. |
| **Scripts enviados** | Definir limite, armazenamento, versionamento, checagem de integridade, isolamento para testes, aprovação, ativação e erros de validação. |
| **Auditoria/retry** | Refinar campos, eventos e retenção de logs; modelar `job_attempt` antes de permitir `max_retries > 0`. |
| **Assinaturas por evento** | Refinar contratos, permissões, persistência e sequência; periodicidade de conteúdo permanece backlog separado. |
| **Governança** | Autoridade formal de aprovação de mudanças ainda aberta; gates documentais indicados como `BLOCKED`. |

**Regra de leitura:** quando uma seção antiga estabelecer valor conflitante com emenda posterior expressa, identificar a decisão posterior (por exemplo, **50.000.000 bytes** para mídia fracionada, revisado em 21/10/2026). Se não houver uma decisão explícita de precedência, **manter como pendência**, sem apresentar uma solução como norma aprovada.

## 14. Rastreabilidade compacta

Este documento é **autossuficiente quanto às regras resumidas**. Os caminhos abaixo identificam as fontes **contidas dentro do arquivo legado consolidado**, não links obrigatórios para documentos separados:

| Assunto consolidado | Principais origens históricas |
|---|---|
| Propósito, escopo e exclusões | `docs/01-Project_Charter/charter/README.md`, `docs/01-Project_Charter/scope/README.md`, `docs/especificacao/requisitos/mvp.md` |
| Arquitetura, bots, segurança | `docs/adr/telegram/multiplos-clientes-isolados.md`, `docs/adr/seguranca/menor-privilegio-e-autorizacao.md`, `docs/seguranca-acessos.md` |
| Contatos e solicitações | `docs/01-Project_Charter/scope/flows/FLW-006-start-registro-de-contato-e-apresentacao-do-menu.md`, `docs/solicitacoes-acesso.md` |
| Administração e validação de scripts | `docs/guia-usuario.md`, `docs/especificacao/contratos/execucao-script.md`, `docs/especificacao/configuracao/modelo-configuracao.md` |
| Comandos e ações | `docs/especificacao/contratos/comando-interno.md`, `docs/01-Project_Charter/scope/flows/FLW-007-cadastro-administrativo-e-execucao-de-acao-persistente.md` |
| Telegram e filas | `docs/especificacao/modulos/telegram.md`, `docs/telegram-operacao-2026.md`, `docs/especificacao/persistencia/claim-concorrente-filas.md` |
| Jobs e shutdown | `docs/especificacao/contratos/job.md`, `docs/especificacao/contratos/processador-assincrono.md`, `docs/adr/runtime/encerramento-de-jobs.md` |
| Mídia | `docs/especificacao/contratos/ingestao-midia-messagepack.md`, `docs/especificacao/contratos/ingestao-midia-fracionada.md`, `docs/especificacao/contratos/transmissao-midia.md` |
| Scheduler e assinatura de eventos | `docs/especificacao/fluxos/monitoramento-agendado.md`, `docs/charter/objectives/OBJ-009-assinatura-e-entrega-de-eventos-telegram.md` |
| Modelo relacional e bloqueios | `docs/especificacao/persistencia/tabelas-estado-operacional.md`, `docs/especificacao/persistencia/relacionamentos-estado-operacional.md`, `docs/pendencias.md` |

---

**Situação da síntese:** conteúdo reorganizado para servir como **referência única** à criação de novos documentos. A síntese reduz redundância **sem transformar propostas e lacunas em decisões finalizadas**.
