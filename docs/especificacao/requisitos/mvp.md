# Escopo do primeiro MVP

**ID:** REQ-0001  
**Status:** refined

## Objetivo

Delimitar o primeiro MVP para impedir implementação prematura de processadores complexos.

## Dependências

- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)

## Capacidades obrigatórias

### Core

- Sabiá Core;
- múltiplos clientes;
- Script Registry;
- Script Executor;
- Job Manager;
- Scheduler;
- Alert Manager;
- Telegram Long Polling;
- autorização por Telegram User ID;
- logs;
- configuração YAML;
- graceful shutdown;
- Unix socket local de ingestão simples de mídia com MessagePack, limitado a 20 MB;
- segundo Unix socket para ingestão fracionada em chunks de até 5 MB, opcional para arquivos de até 20 MB e obrigatório acima desse limite;
- persistência de mídia em PostgreSQL por `media_id`;
- fila persistente de transmissão de mídia;
- integração com systemd;
- processo executado como usuário `sabia`.

### Cliente Bioma

MVP:
- `/start`;
- `/help`;
- `/status`;
- `/cpu`;
- `/memory`;
- `/disk`;
- `/uptime`;
- `/run`.

Catálogo previsto para evolução:
- `/network`;
- `/processes`;
- `/services`;
- `/jobs`;
- `/job <id>`.

### Cliente Tools

MVP:
- `/help`;
- `/jobs`;
- `/job`.

Previstos, mas fora do primeiro processamento complexo:
- `/image`;
- `/video`;
- `/audio`;
- `/convert`;
- `/resize`;
- `/compress`;
- `/cancel`.

### Cliente Alerts

MVP:
- `/status`.

Previstos:
- `/checks`;
- `/check <nome>`;
- `/alerts`.

## Scripts iniciais

Scripts simples previstos para validar a arquitetura:
- system/status.sh;
- system/cpu.sh;
- system/memory.sh;
- system/disk.sh;
- system/uptime.sh;
- system/network.sh;
- monitoring/disk-check.sh;
- monitoring/service-check.sh.

## Primeira prova funcional

`disk-check` executado pelo Scheduler e notificado por Sabiá Alerts quando houver mudança de estado.

## Fora do MVP

Não implementar ainda os processadores complexos de imagem, vídeo e áudio listados como evolução. A infraestrutura de ingestão, persistência e transporte de mídia, porém, faz parte do MVP para permitir que processadores externos utilizem o Sabiá sem alterar o Core.

## Critérios de aceite

- fluxo Telegram → Bioma → comando → script → resultado → Telegram é possível;
- fluxo Scheduler → disk-check → estado relevante → Alerts → Telegram é possível;
- nenhum desses fluxos exige shell arbitrário nem porta pública de webhook;
- produtor local autorizado consegue persistir mídia integral ou fracionada associada a `request_id`;
- arquivo fracionado pode ser reconstruído por sequência sem alocar o conteúdo completo em memória;
- transmissão de mídia pendente sobrevive a restart do serviço.
