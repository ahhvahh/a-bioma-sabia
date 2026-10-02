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

Cada script cadastrado possui, no mínimo:

- identificador lógico;
- `path`;
- `timeout`;
- indicação explícita de repetibilidade quando retry automático for permitido;
- `max_retries`, com padrão `0`.

Detalhes de working directory, ambiente e limites de saída dependem de CTR-0002.

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
- lembrete quando aplicável.

### logging

Deve permitir configuração de nível e saída, preservando o contrato de não registrar segredos.

## Validação

A configuração completa é validada antes de iniciar:

- clientes Telegram;
- scheduler;
- workers;
- processamento de comandos.

Configuração ausente, inválida, campo obrigatório ausente, segredo referenciado inexistente ou valor fora do domínio faz o processo falhar no startup.

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
- jobs possuem limites configuráveis;
- SQLite possui caminho conhecido e schema versionado.
