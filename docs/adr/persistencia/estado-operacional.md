# Persistência do estado operacional

![ADR](https://img.shields.io/badge/ADR-ADR--0009-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

Jobs, requisições, respostas Telegram e alertas possuem estado operacional que não pode ser perdido apenas porque o processo do Sabiá foi reiniciado.

## Problema

Definir o mecanismo oficial de persistência e o comportamento mínimo de recuperação após reinicialização.

## Restrições

- a solução deve ser local, simples e de baixo consumo;
- requisições pendentes precisam sobreviver ao reinício;
- respostas ainda não entregues precisam continuar disponíveis para envio;
- jobs precisam manter correlação com sua origem e destino de resposta;
- o estado anterior necessário ao controle de alertas deve sobreviver ao reinício;
- a persistência não deve transformar operações longas em transações longas de banco.

## Opções consideradas

### Estado apenas em memória

Não atende à necessidade de recuperar requisições e respostas pendentes após reinicialização.

### SQLite local

Mantém o estado operacional no próprio servidor, sem exigir um serviço de banco de dados externo.

## Decisão

Adotar **SQLite** como banco de dados local oficial para o estado operacional persistente do Sabiá.

Devem ser persistidos, quando aplicáveis:

- requisições recebidas e seu estado de processamento;
- correlação entre requisição, cliente, usuário, chat, mensagem e job;
- jobs e estados necessários para identificar trabalho pendente;
- respostas ou mensagens ainda pendentes de entrega;
- estado anterior necessário à avaliação de alertas;
- parâmetros operacionais que precisem sobreviver ao reinício.

Ao iniciar, o Sabiá deve consultar o banco e identificar requisições, jobs e mensagens pendentes. Respostas que já estejam prontas mas ainda não tenham sido entregues devem voltar para a etapa de envio ao destino correspondente.

A política para um job que estava efetivamente em execução no instante da interrupção não é definida por este ADR; ela permanece responsabilidade do contrato de jobs e da política de encerramento.

## Justificativa

SQLite atende ao volume e ao perfil local do Sabiá sem introduzir um serviço de banco adicional e permite recuperar o estado necessário depois de reinicializações.

## Consequências

- o banco SQLite passa a ser parte do estado operacional do serviço;
- operações de persistência devem usar transações curtas;
- fila e respostas pendentes podem ser reconstruídas a partir do banco;
- o serviço não deve considerar uma resposta definitivamente concluída enquanto o estado de entrega correspondente não estiver registrado;
- retenção, localização física do arquivo e evolução de schema ficam na especificação de configuração/persistência, sem reabrir esta decisão arquitetural;
- comportamento de jobs interrompidos continua dependente de ADR-0010 e CTR-0003.

## Dependências

- [ADR-0005 — Telegram Long Polling](../telegram/long-polling.md)
- [ADR-0006 — Jobs assíncronos](../processamento/jobs-assincronos.md)
- [ADR-0007 — Scheduler e alertas orientados a estado](../monitoramento/scheduler-alertas-estado.md)

## Critérios de validação

- SQLite é usado como armazenamento operacional persistente;
- uma requisição pendente continua identificável depois de reiniciar o serviço;
- uma resposta pronta e ainda não entregue volta a ficar disponível para envio;
- o estado anterior necessário a alertas pode ser recuperado;
- jobs interrompidos não têm sua política inferida por este ADR.
