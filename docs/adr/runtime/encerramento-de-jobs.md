# Política de encerramento de jobs

![ADR](https://img.shields.io/badge/ADR-ADR--0010-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

Ao receber SIGTERM ou SIGINT, o Sabiá deve encerrar de forma controlada sem perder requisições, respostas pendentes ou contexto necessário à comunicação com Telegram.

## Problema

Definir o comportamento de jobs, respostas e clientes Telegram durante o graceful shutdown.

## Restrições

- nenhum novo trabalho deve ser aceito após o início do shutdown;
- requisições e respostas pendentes são persistidas em SQLite conforme ADR-0009;
- respostas produzidas pertencem ao cliente Telegram e ao destino que originaram a requisição;
- todos os clientes Telegram ativos devem ser avisados antes do encerramento;
- jobs em execução recebem um período configurado para finalizar.

## Opções consideradas

### Cancelar imediatamente

Reduz o tempo de shutdown, mas interrompe respostas potencialmente próximas da conclusão.

### Aguardar indefinidamente

Preserva trabalho, mas impede um encerramento previsível.

### Aguardar até timeout configurado

Permite concluir trabalho pendente dentro de um limite operacional conhecido.

## Decisão

Ao iniciar o shutdown:

1. o Sabiá para de aceitar novas requisições e novos jobs;
2. todos os clientes Telegram ativos são colocados em estado de encerramento;
3. cada cliente ativo envia aviso de encerramento aos seus destinos autorizados conhecidos e registrados no estado operacional;
4. schedulers deixam de iniciar novas execuções;
5. jobs em `queued` permanecem persistidos para o próximo startup;
6. jobs em `running` recebem até `service.shutdown_timeout` para finalizar;
7. respostas concluídas durante esse período são enviadas pelo mesmo cliente Telegram e ao mesmo destino correlacionado à requisição;
8. respostas prontas que não puderem ser entregues antes do encerramento permanecem persistidas como pendentes;
9. ao atingir o timeout, jobs ainda `running` são cancelados e registrados como `failed` com motivo `shutdown_timeout`;
10. o serviço persiste o estado final necessário, fecha recursos e encerra.

`service.shutdown_timeout` é parâmetro obrigatório e positivo da configuração.

## Justificativa

A política preserva a entrega de respostas sempre que possível, limita o tempo máximo de encerramento e mantém recovery seguro para trabalho e mensagens pendentes.

## Consequências

- o shutdown pode aguardar até o timeout configurado;
- clientes precisam manter destinos conhecidos suficientes para o aviso operacional;
- respostas sempre preservam a correlação com o cliente e destino de origem;
- jobs interrompidos pelo limite não são reexecutados automaticamente;
- jobs ainda em fila continuam disponíveis após reinício.

## Dependências

- [ADR-0006 — Jobs assíncronos](../processamento/jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../persistencia/estado-operacional.md)

## Critérios de validação

- novos jobs deixam de ser aceitos no início do shutdown;
- todos os clientes Telegram ativos tentam enviar aviso de encerramento;
- job `queued` permanece recuperável;
- job `running` pode concluir dentro do timeout;
- resposta concluída é enviada pelo cliente e destino corretos;
- resposta não entregue permanece pendente no SQLite;
- job que ultrapassa o timeout termina como `failed/shutdown_timeout`.
