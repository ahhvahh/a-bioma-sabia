# Modelo de configuração

**ID:** CFG-0001  
**Status:** refinement

## Objetivo

Definir a fonte, estrutura e regras de validação da configuração operacional do Sabiá.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](../../adr/telegram/multiplos-clientes-isolados.md)
- [ADR-0004 — Registro explícito de scripts](../../adr/execucao/registro-explicito-de-scripts.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../../adr/seguranca/menor-privilegio-e-autorizacao.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)
- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](../../adr/processamento/processadores-assincronos-registrados.md)
- [ADR-0012 — Ingestão persistente de mídia por Unix socket](../../adr/processamento/ingestao-midia-socket-messagepack.md)
- [ADR-0013 — Dois canais de ingestão de mídia e upload fracionado](../../adr/processamento/ingestao-midia-fracionada.md)

## Fonte principal

A configuração não secreta do MVP fica em um único arquivo:

`/etc/sabia/sabia.yaml`

O serviço lê esse arquivo apenas no startup. Alterações exigem restart.

## Segredos

Segredos não são armazenados diretamente no YAML versionável.

Campos de segredo usam referência, por exemplo:

`token_env: SABIA_BIOMA_TOKEN`

A variável de ambiente referenciada deve existir no startup. Não existe sobreposição genérica entre YAML e ambiente: YAML é a fonte de configuração operacional; variáveis de ambiente são usadas para valores secretos explicitamente referenciados.

## Estrutura

O arquivo deve possuir as seções:

- `service`;
- `database`;
- `telegram.clients`;
- `scripts`;
- `processors`;
- `media_ingest`;
- `media_chunk_ingest`;
- `jobs`;
- `schedules`;
- `logging`.

### service

Campos mínimos:

- `shutdown_timeout`: duração positiva e obrigatória.

### database

Campos mínimos:

- `path`: caminho do arquivo SQLite.

O caminho padrão é:

`/var/lib/sabia/sabia.db`

### telegram.clients

Cada cliente possui:

- `enabled`;
- `token_env`;
- `allowed_users`;
- `allowed_chats` opcional;
- comandos/operações habilitados.

Clientes iniciais:

- `bioma`;
- `tools`;
- `alerts`.

### scripts

Cada script cadastrado possui:

- identificador lógico;
- `path`: caminho absoluto do executável ou script;
- `timeout`: duração positiva;
- `interpreter`: caminho absoluto opcional de interpretador explicitamente cadastrado;
- `working_directory`: diretório absoluto opcional;
- `allowed_environment`: lista de nomes de variáveis que podem ser herdadas;
- indicação explícita de repetibilidade quando retry automático for permitido;
- `max_retries`, com padrão `0`.

Quando `working_directory` estiver ausente, o executor utiliza o diretório que contém `path`.

Quando `interpreter` estiver ausente, `path` é executado diretamente.

`allowed_environment` vazio significa não herdar variáveis do ambiente do Sabiá.

O limite de captura definido por CTR-0002 é fixo no MVP em 1 MiB para stdout e 1 MiB para stderr por execução.

### processors

Processadores assíncronos podem representar:

- script Bash;
- aplicação/executável;
- serviço acessível por socket.

Cada entrada precisa possuir identificador lógico e tipo de processador.

A estrutura concreta por tipo de transporte ainda está `BLOCKED` e precisa definir, no mínimo:

- como localizar/invocar o processador cadastrado;
- timeout da requisição;
- mecanismo de envio do comando;
- mecanismo de recebimento dos eventos `loading` e `finally`;
- parâmetros necessários ao framing do canal de controle.

Nenhum endereço, caminho ou socket pode ser substituído por valor vindo do comando remoto.

### media_ingest

Configura o socket local exclusivo de ingestão de mídia.

Campos necessários:

- `socket_path`: caminho absoluto do Unix socket;
- parâmetros de ownership/permissão local;
- `max_payload_bytes`: limite máximo aceito por upload MessagePack; no MVP o valor normativo é `20000000` bytes (20 MB decimais).

O caminho e ownership/grupo/modo permanecem `BLOCKED`. O limite não é livremente configurável no MVP: valores diferentes de `20000000` são inválidos.

Não existe endereço TCP para esta interface.

### media_chunk_ingest

Configura o segundo Unix socket, exclusivo para ingestão fracionada.

Campos necessários:

- `socket_path`: caminho absoluto, diferente de `media_ingest.socket_path`;
- parâmetros de ownership/permissão local;
- `max_chunk_bytes`: no MVP deve ser exatamente `5000000` bytes.

Arquivos acima do limite do canal simples usam obrigatoriamente este canal. Arquivos menores ou iguais a 20 MB também podem usá-lo. A abertura gera `media_id` e a sequência é estrita; permanecem `BLOCKED` somente a completude e o limite total do arquivo lógico em CTR-0009.

Não existe endereço TCP para esta interface.

### jobs

Campos obrigatórios:

- `max_workers`: inteiro maior que zero;
- `max_pending`: inteiro maior que zero.

### schedules

Cada agendamento possui, no mínimo:

- identificador;
- operação/script cadastrado;
- `interval`: duração positiva obrigatória entre disparos;
- habilitação;
- política de relatório;
- `reminder_interval`: duração positiva opcional para lembretes de estado degradado inalterado.

No MVP, `interval` é a única forma de periodicidade. Expressões cron, calendários e horários absolutos não são aceitos.

`reminder_interval` ausente significa que o agendamento não envia lembretes periódicos para estado inalterado. Quando presente, aplica-se somente a `WARNING`, `CRITICAL` e `UNKNOWN`; mudança de estado reinicia a contagem e `OK` encerra lembretes.

### logging

Deve permitir configuração de nível e saída, preservando o contrato de não registrar segredos.

## Validação

A configuração completa é validada antes de iniciar:

- clientes Telegram;
- scheduler;
- workers;
- processamento de comandos.

Configuração ausente, inválida, campo obrigatório ausente, segredo referenciado inexistente ou valor fora do domínio faz o processo falhar no startup.

Para agendamentos, `interval` é obrigatório e deve ser uma duração positiva. `reminder_interval`, quando presente, também deve ser uma duração positiva. Configuração cron é inválida no MVP.

Para scripts:

- `path` deve ser absoluto;
- `interpreter`, quando presente, deve ser absoluto;
- `working_directory`, quando presente, deve ser absoluto;
- `timeout` deve ser positivo;
- nomes em `allowed_environment` devem ser válidos e não duplicados.

O Sabiá não deve iniciar parcialmente com configuração inválida.

## Reload

Não existe reload dinâmico no primeiro MVP.

Qualquer alteração de configuração exige reinicialização controlada do serviço.

## SQLite

O arquivo SQLite é criado/aberto no caminho configurado.

O schema deve possuir versão persistida para permitir migrações futuras. Migrações compatíveis com a versão do binário são executadas antes de liberar os demais módulos no startup. Falha de migração bloqueia a inicialização.

## Retenção

Políticas de retenção não devem apagar:

- jobs ainda não finalizados;
- respostas ainda não entregues;
- estado atual necessário aos alertas.

A duração de retenção de histórico finalizado poderá ser adicionada como parâmetro sem alterar o contrato base.

## Critérios de aceite

- existe uma única fonte YAML operacional no MVP;
- segredos são referenciados e não gravados diretamente no YAML;
- configuração inválida impede startup;
- não existe reload dinâmico;
- definição de script contém os campos necessários a CTR-0002;
- existe seção `processors` para processadores assíncronos registrados;
- existe seção `media_ingest` para o Unix socket MessagePack;
- `media_ingest.socket_path` deve ser absoluto;
- `media_ingest.max_payload_bytes` deve ser exatamente `20000000` no MVP;
- existe seção `media_chunk_ingest` com socket distinto;
- `media_chunk_ingest.max_chunk_bytes` deve ser exatamente `5000000` no MVP;
- o schema de `processors` e os valores de segurança de `media_ingest` precisam ser refinados antes de retornar CFG-0001 a `refined`;
- scripts não recebem ambiente completo por herança implícita;
- jobs possuem limites configuráveis;
- `interval` é obrigatório, positivo e é a única periodicidade aceita no MVP;
- cron não é aceito no MVP;
- `reminder_interval` é opcional, positivo quando presente e ausente significa sem lembretes periódicos;
- SQLite possui caminho conhecido e schema versionado.
