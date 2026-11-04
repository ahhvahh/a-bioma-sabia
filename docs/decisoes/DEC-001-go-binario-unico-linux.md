<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# DEC-001 — Go, binário único e serviço Linux

**ID:** `DEC-001`  
**ID histórico:** `ADR-0001`  
**Estado documental de origem:** `refined`  
**Versão histórica registrada:** `1`  
**Implementação:** não verificada nesta revisão

## Decisão

Implementar inicialmente em Go e distribuir o serviço principal como um único binário chamado `sabia`, executável pelo systemd.

## Contexto e problema

O Sabiá deve operar em Debian/Linux, com baixo consumo de recursos, instalação simples e integração natural com systemd.

Definir a tecnologia inicial e a forma de distribuição do serviço.

## Restrições

- execução em Linux;
- evitar dependências desnecessárias;
- gerar um único executável principal;
- o serviço não deve depender de runtime externo para iniciar.

## Alternativas consideradas

### Go com binário único

Atende explicitamente ao escopo inicial e simplifica distribuição do serviço.

### Outra tecnologia ou múltiplos artefatos executáveis

Não corresponde à tecnologia inicial definida para o projeto.

## Justificativa

Esta é a tecnologia e a forma de distribuição definidas para o primeiro ciclo do projeto.

## Consequências

- módulos do serviço compartilham o mesmo processo principal;
- dependências devem ser mantidas pequenas;
- scripts e ferramentas externas continuam sendo executáveis independentes;
- a implementação deve permitir graceful shutdown por SIGTERM e SIGINT.

## Critérios de validação

- o artefato principal gerado é um único binário `sabia`;
- o serviço pode ser gerenciado pelo systemd;
- a aplicação não exige execução como root.

## Dependências

- nenhuma.

## Limitações da revisão

Os critérios são verificáveis, mas não há prova de binário compilado ou serviço systemd ativo nesta revisão.

A revisão documental não demonstra implementação, execução de testes ou aprovação de novas regras.

## Fonte e rastreabilidade

Documento histórico preservado: [`ADR-0001` original](../adr/runtime/go-binario-unico.md). O identificador e a versão original foram mantidos para rastrear a decisão, sem renumerar outros ADRs.
