<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# DEC-004 — Registro explícito de scripts

**ID:** `DEC-004`  
**ID histórico:** `ADR-0004`  
**Estado documental de origem:** `refined`  
**Versão histórica registrada:** `1`  
**Implementação:** não verificada nesta revisão

## Decisão

Toda execução de script originada pelo Sabiá deve resolver um identificador em um Script Registry previamente configurado. Nenhum caminho ou comando de shell arbitrário recebido do Telegram pode ser executado.

## Contexto e problema

O MVP usa scripts locais para consultar o sistema e executar verificações.

Permitir automação sem transformar mensagens Telegram em comandos arbitrários do sistema operacional.

## Restrições

- caminhos arbitrários informados pelo usuário são proibidos;
- texto Telegram não pode ser concatenado em `bash -c`;
- cada script possui identificador cadastrado e timeout;
- o usuário remoto conhece apenas o identificador lógico permitido.

## Alternativas consideradas

### Execução livre de shell

Incompatível com o requisito de segurança.

### Registro explícito de scripts

Restringe execução a operações previamente aprovadas.

## Justificativa

Reduz a superfície de injeção e mantém a lista de operações controlável.

## Consequências

- novos scripts precisam ser cadastrados antes de uso;
- o executor recebe uma definição já resolvida pelo registro;
- o contrato precisa capturar stdout, stderr, exit code, duração e timeout.

## Critérios de validação

- `/run server-status` pode resolver um script cadastrado;
- `/run /tmp/x.sh` não executa um caminho não cadastrado;
- mensagem Telegram nunca é usada como corpo de shell.

## Dependências

- [ADR-0003 — Core independente do Telegram](DEC-003-core-independente-telegram.md)

## Limitações da revisão

O cadastro de scripts está documentado, mas há divergência a resolver entre configuração YAML e catálogo persistente em [pendências de persistência](../persistencia/pendencias.md).

A revisão documental não demonstra implementação, execução de testes ou aprovação de novas regras.

## Fonte e rastreabilidade

Documento histórico preservado: [`ADR-0004` original](https://github.com/ahhvahh/a-bioma-sabia/blob/f898239c4519b1c8fd21327adf7a8407a1bbeaf7/docs/adr/execucao/registro-explicito-de-scripts.md). O identificador e a versão original foram mantidos para rastrear a decisão, sem renumerar outros ADRs.
