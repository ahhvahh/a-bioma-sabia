# Múltiplos clientes Telegram isolados

![ADR](https://img.shields.io/badge/ADR-ADR--0002-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

O Sabiá deve atender responsabilidades diferentes: integração com o ecossistema Bioma, ferramentas de processamento e alertas automáticos.

## Problema

Definir se essas responsabilidades compartilham uma única identidade Telegram ou operam como clientes independentes.

## Restrições

Cada cliente precisa possuir token, nome, usuários, chats, comandos, scripts e permissões próprios.

## Opções consideradas

### Cliente Telegram único

Mistura autorização e catálogo de comandos de responsabilidades distintas.

### Múltiplos clientes independentes

Mantém isolamento lógico e permite evolução separada.

## Decisão

O Sabiá Core operará múltiplos clientes Telegram independentes. O conjunto inicial é `bioma`, `tools` e `alerts`.

## Justificativa

A separação atende ao requisito explícito de isolamento entre clientes Telegram.

## Consequências

- cada cliente possui configuração e autorização próprias;
- comandos e scripts não são implicitamente compartilhados;
- falha ou desativação de um cliente não deve alterar o catálogo dos demais;
- o Core não pode pressupor um bot específico.

## Dependências

- nenhuma.

## Critérios de validação

- é possível configurar tokens distintos;
- a autorização é avaliada no contexto do cliente;
- um comando não fica disponível em outro cliente sem cadastro explícito.
