<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# DEC-002 — Múltiplos clientes Telegram isolados

**ID:** `DEC-002`  
**ID histórico:** `ADR-0002`  
**Estado documental de origem:** `refined`  
**Versão histórica registrada:** `1`  
**Implementação:** não verificada nesta revisão

## Decisão

O Sabiá Core operará múltiplos clientes Telegram independentes. O conjunto inicial é `bioma`, `tools` e `alerts`.

## Contexto e problema

O Sabiá deve atender responsabilidades diferentes: integração com o ecossistema Bioma, ferramentas de processamento e alertas automáticos.

Definir se essas responsabilidades compartilham uma única identidade Telegram ou operam como clientes independentes.

## Restrições

Cada cliente precisa possuir token, nome, usuários, chats, comandos, scripts e permissões próprios.

## Alternativas consideradas

### Cliente Telegram único

Mistura autorização e catálogo de comandos de responsabilidades distintas.

### Múltiplos clientes independentes

Mantém isolamento lógico e permite evolução separada.

## Justificativa

A separação atende ao requisito explícito de isolamento entre clientes Telegram.

## Consequências

- cada cliente possui configuração e autorização próprias;
- comandos e scripts não são implicitamente compartilhados;
- falha ou desativação de um cliente não deve alterar o catálogo dos demais;
- o Core não pode pressupor um bot específico.

## Critérios de validação

- é possível configurar tokens distintos;
- a autorização é avaliada no contexto do cliente;
- um comando não fica disponível em outro cliente sem cadastro explícito.

## Dependências

- nenhuma.

## Limitações da revisão

Os clientes iniciais `bioma`, `tools` e `alerts` são previstos. O cadastro e a autorização dinâmica continuam sujeitos ao modelo persistente em [pendências de persistência](../persistencia/pendencias.md).

A revisão documental não demonstra implementação, execução de testes ou aprovação de novas regras.

## Fonte e rastreabilidade

Documento histórico preservado: [`ADR-0002` original](../adr/telegram/multiplos-clientes-isolados.md). O identificador e a versão original foram mantidos para rastrear a decisão, sem renumerar outros ADRs.
