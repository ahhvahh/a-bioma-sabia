# Política de encerramento de jobs

![ADR](https://img.shields.io/badge/ADR-ADR--0010-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

## Contexto

Ao receber SIGTERM ou SIGINT, o Sabiá deve parar de aceitar novos jobs, parar schedulers, finalizar workers, salvar estado e encerrar recursos.

## Problema

Definir o destino de jobs que já estão em execução durante o graceful shutdown.

## Restrições

O escopo original admite cancelar ou concluir operações "conforme política", sem definir a política.

## Opções consideradas

### Concluir jobs em andamento

Aumenta o tempo de shutdown.

### Cancelar jobs em andamento

Encerra mais rápido, mas exige semântica de cancelamento e recuperação.

### Política configurável

Permite escolher por operação ou instalação, mas adiciona configuração.

## Decisão

**BLOCKED.** A política ainda não foi escolhida.

## Justificativa

O comportamento afeta integridade do job, tempo de shutdown e experiência do usuário.

## Consequências

FLW-0004 e os critérios finais de MOD-0004 permanecem em `refinement`.

## Dependências

- [ADR-0006 — Jobs assíncronos](../processamento/jobs-assincronos.md)
- [ADR-0009 — Persistência do estado operacional](../persistencia/estado-operacional.md)

## Critérios de validação

- definir política de jobs em `queued`;
- definir política de jobs em `running`;
- definir tempo máximo de shutdown, se existir;
- definir estado final registrado para operação interrompida.
