<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# DEC-007 — Scheduler e alertas orientados a estado

**ID:** `DEC-007`  
**ID histórico:** `ADR-0007`  
**Estado documental de origem:** `refined`  
**Versão histórica registrada:** `1`  
**Implementação:** não verificada nesta revisão

## Decisão

O Alert Manager compara o estado atual com o estado anterior e publica alerta em transições relevantes, recuperação e lembretes configurados.

## Contexto e problema

O Sabiá Alerts deve emitir mensagens automaticamente a partir de verificações periódicas, evitando repetição excessiva.

Definir como verificações agendadas geram alertas relevantes.

## Restrições

- scheduler executa tarefas cadastradas;
- monitoramentos usam estados `OK`, `WARNING`, `CRITICAL`, `UNKNOWN`;
- transições relevantes geram alerta;
- estado repetido não deve gerar alerta imediato;
- lembretes periódicos podem ser configurados;
- recuperação deve ser comunicada.

## Alternativas consideradas

### Alertar a cada execução

Gera mensagens repetitivas.

### Alertar por transição de estado

Gera mensagens quando há mudança relevante, com lembretes controlados.

## Justificativa

Atende ao requisito de reduzir ruído sem ocultar mudanças de saúde do sistema.

## Consequências

- o estado anterior precisa estar disponível ao avaliar uma checagem;
- o estado entre reinícios é persistido no PostgreSQL conforme ADR-0009;
- scripts de monitoramento devem produzir resultado interpretável.

## Critérios de validação

- `OK → CRITICAL` alerta;
- `CRITICAL → CRITICAL` não alerta imediatamente;
- `CRITICAL → OK` envia recuperação;
- lembrete só ocorre quando configurado.

## Dependências

- [ADR-0004 — Registro explícito de scripts](DEC-004-registro-explicito-scripts.md)
- [ADR-0009 — Persistência do estado operacional](DEC-009-postgresql-estado-operacional.md)

## Limitações da revisão

A persistência do estado de alertas é prevista, mas a correlação de notificações sem `request_id` e a atualização de `last_notified_at` continuam pendentes na [documentação de persistência](../persistencia/pendencias.md).

A revisão documental não demonstra implementação, execução de testes ou aprovação de novas regras.

## Fonte e rastreabilidade

Documento histórico preservado: [`ADR-0007` original](../adr/monitoramento/scheduler-alertas-estado.md). O identificador e a versão original foram mantidos para rastrear a decisão, sem renumerar outros ADRs.
