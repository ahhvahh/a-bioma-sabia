# Modelo de configuração

**ID:** CFG-0001  
**Status:** refined

## Objetivo

Definir a fonte, estrutura e regras de validação da configuração operacional do Sabiá.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](../../adr/telegram/multiplos-clientes-isolados.md)
- [ADR-0004 — Registro explícito de scripts](../../adr/execucao/registro-explicito-de-scripts.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../../adr/seguranca/menor-privilegio-e-autorizacao.md)
- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0010 — Política de encerramento de jobs](../../adr/runtime/encerramento-de-jobs.md)

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

### jobs

Campos obrigatórios:

- `max_workers`: inteiro maior que zero;
- `max_pending`: inteiro maior que zero.

### schedules

Cada agendamento possui, no mínimo:

- identificador;
- operação/script cadastrado;
- periodicidade;
- habilitação;
- política de relatório;
- `reminder_interval`: duração positiva opcional para lembretes de estado degradado inalterado.

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

Para agendamentos, `reminder_interval`, quando presente, deve ser uma duração positiva.

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
- scripts não recebem ambiente completo por herança implícita;
- jobs possuem limites configuráveis;
- `reminder_interval` é opcional, positivo quando presente e ausente significa sem lembretes periódicos;
- SQLite possui caminho conhecido e schema versionado.
