<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# CONTR-004 — Auditoria e logs

**ID documental:** `CONTR-004` · **ID histórico:** `CTR-0004`  
**Estado da especificação de origem:** `refinement` · **Implementação:** não verificada  
**Tipo:** contrato técnico do Sabiá; a classificação `REVISADO` indica revisão documental, não aprovação funcional nem código executado.

## Objetivo

Padronizar informações mínimas de observabilidade e auditoria sem registrar segredos.

## Dependências

- [ARQ-006 (MOD-0006) — Segurança e autorização](../arquitetura/ARQ-006-seguranca-autorizacao.md)
- [DEC-008 (ADR-0008) — Menor privilégio e autorização explícita](../decisoes/DEC-008-menor-privilegio-autorizacao.md)

## Tipo

`evento`

## Entrada

Eventos operacionais do serviço.

## Saída

Logs estruturados devem poder representar:
- timestamp;
- level;
- component;
- client;
- command;
- job_id;
- message.

Auditoria de comandos deve poder representar:
- timestamp;
- client;
- telegram_user_id;
- chat_id;
- command;
- job_id;
- resultado;
- tempo.

Campos não aplicáveis podem estar ausentes, mas o significado não deve mudar.

## Erros

Falha de logging não autoriza revelar segredo nem transformar dado sensível em fallback textual.

## Regras e restrições

Nunca registrar:
- tokens;
- senhas;
- segredos;
- credenciais;
- BLOBs de mídia;
- campo `data` de CTR-0008.

A saída inicial é stdout/stderr para integração natural com systemd/journalctl.

## Compatibilidade

O formato deve permanecer estruturado mesmo quando novos componentes forem adicionados.

## BLOCKED

Antes de retornar a `refined`, definir:

- tipos dos campos estruturados;
- campos obrigatórios por família de evento;
- identificador/nome estável de cada evento auditável;
- resultado/estado permitido para cada família;
- tratamento de identificadores potencialmente sensíveis, além da proibição já definida de segredos e BLOBs.

## Critérios de aceite

- eventos incluem contexto suficiente para correlação;
- segredos são omitidos;
- conteúdo binário de mídia nunca é serializado para logs/auditoria;
- saída é consumível por journalctl através de stdout/stderr.

## Limitações da revisão

**BLOQUEADO na origem:** os pontos identificados em `BLOCKED` permanecem sem decisão. Não interpretar o conteúdo como um contrato de auditoria integralmente aprovado.

## Rastreabilidade

Fonte integral: [`CTR-0004` — Auditoria e logs](../especificacao/contratos/auditoria-logs.md). As entradas, saídas, estados, erros e restrições anteriores foram preservados; referências relativas foram adaptadas à estrutura canônica.
