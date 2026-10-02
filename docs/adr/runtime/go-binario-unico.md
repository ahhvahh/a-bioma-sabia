# Go, binário único e serviço Linux

![ADR](https://img.shields.io/badge/ADR-ADR--0001-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-1-6e7781?style=flat-square)

## Contexto

O Sabiá deve operar em Debian/Linux, com baixo consumo de recursos, instalação simples e integração natural com systemd.

## Problema

Definir a tecnologia inicial e a forma de distribuição do serviço.

## Restrições

- execução em Linux;
- evitar dependências desnecessárias;
- gerar um único executável principal;
- o serviço não deve depender de runtime externo para iniciar.

## Opções consideradas

### Go com binário único

Atende explicitamente ao escopo inicial e simplifica distribuição do serviço.

### Outra tecnologia ou múltiplos artefatos executáveis

Não corresponde à tecnologia inicial definida para o projeto.

## Decisão

Implementar inicialmente em Go e distribuir o serviço principal como um único binário chamado `sabia`, executável pelo systemd.

## Justificativa

Esta é a tecnologia e a forma de distribuição definidas para o primeiro ciclo do projeto.

## Consequências

- módulos do serviço compartilham o mesmo processo principal;
- dependências devem ser mantidas pequenas;
- scripts e ferramentas externas continuam sendo executáveis independentes;
- a implementação deve permitir graceful shutdown por SIGTERM e SIGINT.

## Dependências

- nenhuma.

## Critérios de validação

- o artefato principal gerado é um único binário `sabia`;
- o serviço pode ser gerenciado pelo systemd;
- a aplicação não exige execução como root.
