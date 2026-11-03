# Scheduler e alertas orientados a estado

![ADR](https://img.shields.io/badge/ADR-ADR--0007-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

O Sabiá Alerts deve emitir mensagens automaticamente a partir de verificações periódicas, evitando repetição excessiva.

## Problema

Definir como verificações agendadas geram alertas relevantes.

## Restrições

- scheduler executa tarefas cadastradas;
- monitoramentos usam estados `OK`, `WARNING`, `CRITICAL`, `UNKNOWN`;
- transições relevantes geram alerta;
- estado repetido não deve gerar alerta imediato;
- lembretes periódicos podem ser configurados;
- recuperação deve ser comunicada.

## Opções consideradas

### Alertar a cada execução

Gera mensagens repetitivas.

### Alertar por transição de estado

Gera mensagens quando há mudança relevante, com lembretes controlados.

## Decisão

O Alert Manager compara o estado atual com o estado anterior e publica alerta em transições relevantes, recuperação e lembretes configurados.

## Justificativa

Atende ao requisito de reduzir ruído sem ocultar mudanças de saúde do sistema.

## Consequências

- o estado anterior precisa estar disponível ao avaliar uma checagem;
- o estado entre reinícios é persistido no PostgreSQL conforme ADR-0009;
- scripts de monitoramento devem produzir resultado interpretável.

## Dependências

- [ADR-0004 — Registro explícito de scripts](../execucao/registro-explicito-de-scripts.md)
- [ADR-0009 — Persistência do estado operacional](../persistencia/estado-operacional.md)

## Critérios de validação

- `OK → CRITICAL` alerta;
- `CRITICAL → CRITICAL` não alerta imediatamente;
- `CRITICAL → OK` envia recuperação;
- lembrete só ocorre quando configurado.
