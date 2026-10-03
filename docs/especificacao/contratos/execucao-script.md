# Execução de script

![CTR](https://img.shields.io/badge/CTR-CTR--0002-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Padronizar a execução controlada de scripts e executáveis cadastrados, incluindo invocação, ambiente, concorrência, timeout, cancelamento e captura limitada de saída.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0004 — Registro explícito de scripts](../../adr/execucao/registro-explicito-de-scripts.md)
- [CFG-0001 — Modelo de configuração](../configuracao/modelo-configuracao.md)

## Tipo

`interface`

## Entrada

Uma execução recebe uma definição já resolvida pelo Script Registry.

Campos conceituais obrigatórios:

- `execution_id: string` — identificador único da tentativa de execução;
- `script_id: string` — identificador lógico previamente cadastrado;
- `executable: string` — caminho absoluto do executável ou script cadastrado;
- `arguments: string[]` — argumentos já validados pela operação;
- `timeout: duration` — limite positivo da execução;
- `allowed_environment: string[]` — nomes das variáveis de ambiente que podem ser herdadas pelo processo filho.

Campos opcionais:

- `interpreter: string` — caminho absoluto de interpretador explicitamente cadastrado;
- `working_directory: string` — diretório de trabalho absoluto.

O usuário remoto nunca fornece `executable`, `interpreter` ou `working_directory`.

## Invocação

### Execução direta

Quando `interpreter` não estiver configurado, o executor inicia diretamente o caminho registrado em `executable` e fornece `arguments` como argumentos do processo.

Não é permitido envolver a chamada automaticamente em:

- `bash -c`;
- `sh -c`;
- outro shell intermediário;
- linha de comando construída a partir de texto recebido do Telegram.

### Interpretador explícito

Quando `interpreter` estiver configurado, o executor inicia diretamente o interpretador cadastrado.

A ordem lógica dos argumentos é:

1. caminho registrado em `executable`;
2. `arguments` validados da operação.

O interpretador também deve possuir caminho absoluto e fazer parte da configuração do script. O solicitante remoto não pode escolher ou substituir o interpretador.

## Diretório de trabalho

Quando `working_directory` estiver configurado, ele é usado como diretório de trabalho da execução.

Quando estiver ausente, o diretório padrão é o diretório que contém o arquivo indicado por `executable`.

O diretório de trabalho nunca é derivado de texto informado pelo usuário remoto.

## Ambiente

O processo filho não herda indiscriminadamente o ambiente completo do Sabiá.

Somente variáveis cujos nomes constem em `allowed_environment` podem ser copiadas do ambiente do serviço para a execução.

Uma variável não listada não deve ser fornecida ao processo por herança implícita.

Segredos usados pelo próprio Sabiá não se tornam automaticamente disponíveis aos scripts.

## Concorrência

Cada solicitação de execução é independente.

O executor:

- não aplica exclusão mútua por `script_id`;
- permite duas ou mais execuções simultâneas do mesmo script;
- não compartilha stdout, stderr, contexto de cancelamento ou estado mutável entre execuções.

Quando uma execução fizer parte de um job, o limite global de jobs concorrentes é controlado por `jobs.max_workers` conforme CTR-0003.

## Uso como processador assíncrono

Quando um script cadastrado é usado como processador assíncrono de CTR-0005, aplicam-se regras adicionais:

- o `request_id` UUID v4 é escrito no `stdin` como uma única linha terminada por `\n`;
- os argumentos já validados continuam sendo fornecidos via argv;
- o `stdout` é reservado ao protocolo JSON Lines de CTR-0005 e não é interpretado como texto livre de resultado;
- o stdout assíncrono é consumido incrementalmente linha a linha e não é acumulado como o campo `stdout` do resultado convencional;
- cada linha de `stdout` deve ser um objeto JSON válido de evento `loading | finally`;
- `stderr` permanece separado e destinado a diagnóstico;
- no MVP, scripts usados como processadores assíncronos possuem timeout cadastrado de `2h`;
- as regras de cancelamento e encerramento do grupo de processos deste contrato continuam válidas.

A execução assíncrona não altera o caminho, interpretador, working directory ou ambiente cadastrado do script.

## Timeout e cancelamento

Cada execução possui contexto independente e cancelável.

O processo iniciado deve pertencer a um grupo de processos próprio da execução.

Quando ocorrer timeout ou cancelamento:

1. impedir continuidade normal daquela execução;
2. encerrar o grupo de processos pertencente à execução;
3. aguardar/recolher o término dos processos controlados pelo executor;
4. retornar resultado indicando `timed_out` ou `cancelled`.

O mecanismo concreto de sinais pode variar na implementação, mas não é permitido deixar deliberadamente o grupo de processos da execução ativo após o executor considerar o cancelamento concluído.

O cancelamento de uma execução não deve afetar outras execuções concorrentes.

## Saída

O resultado conceitual contém:

- `execution_id: string`;
- `exit_code: integer | null`;
- `stdout`;
- `stderr`;
- `stdout_truncated: boolean`;
- `stderr_truncated: boolean`;
- `duration: duration`;
- `timed_out: boolean`;
- `cancelled: boolean`.

`exit_code` pode ser nulo quando o processo não chegou a iniciar ou quando não houver código de saída representável pela execução encerrada.

## Limite de stdout e stderr

Para execução convencional, permanecem os limites de captura definidos abaixo.

Quando o script atua como processador assíncrono CTR-0005, o `stdout` é um stream de protocolo consumido incrementalmente e **não** é acumulado como saída textual capturada. O limite de 1 MiB de stdout não se aplica ao stream CTR-0005. O `stderr` continua sujeito ao limite de diagnóstico deste contrato.

No MVP, cada execução pode reter no máximo:

- **1 MiB de stdout**;
- **1 MiB de stderr**.

Os limites são independentes.

Quando um stream ultrapassar o limite:

- os primeiros 1 MiB permanecem disponíveis no resultado;
- o executor continua drenando o stream para não bloquear o processo;
- o conteúdo excedente é descartado;
- o campo `stdout_truncated` ou `stderr_truncated` correspondente é marcado como `true`.

Truncamento, isoladamente, não altera o exit code e não transforma uma execução bem-sucedida em falha.

## Exit code e monitoramento

Um exit code diferente de zero não é, por si só, erro do executor.

O executor apenas captura e devolve o valor.

Para operações de monitoramento, o Scheduler/Alert Manager pode interpretar:

- `0 = OK`;
- `1 = WARNING`;
- `2 = CRITICAL`;
- `3 = UNKNOWN`.

A interpretação semântica desses códigos pertence ao fluxo de monitoramento, não ao executor.

## Erros

Erros do contrato de execução incluem:

- `script_not_found` — identificador lógico não resolvido;
- `invalid_definition` — definição cadastrada inválida;
- `process_start_failed` — falha ao criar/iniciar o processo;
- `internal_executor_error` — falha interna do executor.

Timeout e cancelamento são condições explícitas de término e devem ser representados pelos campos do resultado.

Um exit code diferente de zero é resultado do processo, não erro de infraestrutura do executor.

## Regras e restrições

- nunca construir shell a partir de mensagem Telegram;
- nunca aceitar caminho arbitrário de executável, interpretador ou diretório do usuário remoto;
- manter stdout e stderr separados;
- quando o script atuar como processador CTR-0005, reservar stdout exclusivamente aos eventos JSON Lines e stdin ao `request_id`;
- aplicar limite de saída por execução;
- não disponibilizar segredos por herança implícita de ambiente;
- permitir execuções simultâneas do mesmo script;
- timeout e cancelamento devem atuar somente sobre o grupo de processos da execução correspondente.

## Compatibilidade

O contrato descreve uma fronteira de execução local e não depende de objetos Telegram.

Executores futuros podem implementar a mesma semântica sem alterar o Command Router, desde que preservem entrada, resultado, timeout, cancelamento e restrições de segurança.

## Critérios de aceite

- execução sem interpretador não usa shell intermediário;
- interpretador, quando utilizado, é explicitamente cadastrado;
- diretório padrão é o diretório do arquivo executado;
- ambiente herdado respeita `allowed_environment`;
- duas chamadas do mesmo script podem executar simultaneamente;
- timeout de uma chamada não cancela outra;
- cancelamento encerra o grupo de processos controlado pela execução;
- stdout e stderr são limitados independentemente a 1 MiB;
- truncamento é sinalizado sem alterar semanticamente o exit code;
- exit code não zero continua disponível ao consumidor;
- `0/1/2/3` só são convertidos em estados de monitoramento pelo Scheduler/Alert Manager.
