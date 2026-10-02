# Persistência do estado operacional em PostgreSQL

![ADR](https://img.shields.io/badge/ADR-ADR--0009-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-3-6e7781?style=flat-square)

## Contexto

Jobs, requisições, respostas Telegram, mídias, transmissões e alertas possuem estado operacional que não pode ser perdido apenas porque o processo do Sabiá foi reiniciado.

O volume de escrita concorrente passou a incluir filas persistentes, múltiplos workers, histórico de estados, entregas pendentes e ingestão de mídia. A persistência também precisa permanecer adequada ao uso concorrente por esses componentes.

## Problema

Definir o banco de dados oficial para o estado operacional persistente do Sabiá e o comportamento mínimo de recuperação após reinicialização.

## Restrições

- requisições pendentes precisam sobreviver ao reinício;
- respostas ainda não entregues precisam continuar disponíveis para envio;
- jobs precisam manter correlação com sua origem e destino de resposta;
- o estado anterior necessário ao controle de alertas deve sobreviver ao reinício;
- mídias e transmissões persistentes devem permanecer recuperáveis;
- operações longas não devem manter transações longas de banco;
- o Sabiá deve possuir isolamento lógico de persistência em relação a outros projetos que utilizem o mesmo servidor de banco.

## Opções consideradas

### SQLite local

Atende a cenários simples e mantém o estado no próprio servidor do Sabiá, mas concentra as escritas concorrentes em um mecanismo local de escritor único.

### PostgreSQL

Oferece persistência transacional adequada ao modelo concorrente do Sabiá e permite que filas, workers, respostas, alertas e mídia compartilhem um mecanismo de persistência com controle explícito de concorrência.

## Decisão

Adotar **PostgreSQL** como banco de dados oficial para o estado operacional persistente do Sabiá.

Não existe fallback normativo para SQLite.

O Sabiá deve utilizar:

- um banco lógico próprio;
- uma role própria de acesso;
- credenciais próprias, fornecidas por mecanismo de segredo;
- schema e migrações controlados pelo próprio Sabiá.

O servidor PostgreSQL pode ser compartilhado com outros projetos, mas tabelas, credenciais e ciclo de migração do Sabiá não são compartilhados implicitamente com eles.

Devem ser persistidos, quando aplicáveis:

- requisições recebidas e seu estado de processamento;
- correlação entre requisição, cliente, usuário, chat, mensagem e job;
- jobs e estados necessários para identificar trabalho pendente;
- respostas ou mensagens ainda pendentes de entrega;
- mídias aceitas, incluindo metadados e conteúdo binário persistente;
- transmissões de mídia pendentes ou em andamento, incluindo `media_id`, cliente, transporte e destino;
- estado anterior necessário à avaliação de alertas;
- parâmetros operacionais que precisem sobreviver ao reinício.

Ao iniciar, o Sabiá deve consultar o PostgreSQL e identificar requisições, jobs, mensagens e transmissões de mídia pendentes. Respostas prontas ainda não entregues voltam para envio. Transmissões de mídia `pending` ou `transmitting` voltam para envio quando o `media_id` persistido existe.

A política para um job que estava efetivamente em execução no instante da interrupção não é definida por este ADR; ela permanece responsabilidade do contrato de jobs e da política de encerramento.

## Justificativa

O modelo atual do Sabiá possui múltiplos componentes concorrentes escrevendo estado operacional. PostgreSQL reduz a limitação estrutural de concorrência de escrita do SQLite e oferece uma base única para transações, filas persistentes, recovery e evolução do modelo de dados.

O uso de banco e role próprios mantém o Sabiá isolado mesmo quando a infraestrutura PostgreSQL é compartilhada.

## Consequências

- PostgreSQL passa a ser dependência operacional do serviço;
- o Sabiá deixa de possuir SQLite como persistência oficial;
- operações de persistência devem usar transações curtas;
- fila, respostas e transmissões de mídia pendentes podem ser reconstruídas a partir do banco;
- o serviço não deve considerar uma resposta definitivamente concluída enquanto o estado de entrega correspondente não estiver registrado;
- estrutura relacional, índices, regras de locking, parâmetros concretos de conexão e evolução de schema ficam na especificação de configuração/persistência;
- comportamento de jobs interrompidos continua dependente de ADR-0010 e CTR-0003.

## Dependências

- [ADR-0005 — Telegram Long Polling](../telegram/long-polling.md)
- [ADR-0006 — Jobs assíncronos](../processamento/jobs-assincronos.md)
- [ADR-0007 — Scheduler e alertas orientados a estado](../monitoramento/scheduler-alertas-estado.md)
- [ADR-0012 — Ingestão persistente de mídia por Unix socket](../processamento/ingestao-midia-socket-messagepack.md)

## Critérios de validação

- PostgreSQL é usado como armazenamento operacional persistente;
- o Sabiá possui banco lógico e role próprios;
- uma requisição pendente continua identificável depois de reiniciar o serviço;
- uma resposta pronta e ainda não entregue volta a ficar disponível para envio;
- uma transmissão de mídia ativa continua identificável e pode ser retomada após restart quando o `media_id` existe;
- mídia confirmada ao produtor por CTR-0008 permanece persistida após restart;
- o estado anterior necessário a alertas pode ser recuperado;
- jobs interrompidos não têm sua política inferida por este ADR.
