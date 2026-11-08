<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# REF-005 — Mapa de rastreabilidade das regras do Sabiá

**Objetivo:** localizar as fontes canônicas dos 14 assuntos do documento consolidado histórico, sem duplicar definições de contratos, políticas e esquemas.

**Situação:** índice documental verificado na branch `devDocumentacao`; **não equivale à comprovação de implementação**. Os **14 assuntos históricos** têm destinos canônicos verificados. Os requisitos adicionais de administração, scripts, aprovação e eventos foram absorvidos como documentos em `refinement`, **sem aprovar contratos nem implementação**.

## Assuntos e fontes canônicas

| Seção histórica | Fonte principal | Referências relacionadas |
|---|---|---|
| 1. Propósito e limites | [ARQ-001-contexto-sabia.md](../arquitetura/ARQ-001-contexto-sabia.md) | [REF-002-escopo-primeiro-mvp.md](REF-002-escopo-primeiro-mvp.md), [DEC-003-core-independente-telegram.md](../decisoes/DEC-003-core-independente-telegram.md) |
| 2. Identidades, isolamento e autorização | [ARQ-006-seguranca-autorizacao.md](../arquitetura/ARQ-006-seguranca-autorizacao.md) | [DEC-008-menor-privilegio-autorizacao.md](../decisoes/DEC-008-menor-privilegio-autorizacao.md) |
| 3. Bots, contatos e solicitações de acesso | [ARQ-008-adaptador-telegram.md](../arquitetura/ARQ-008-adaptador-telegram.md) | [ENT-013-telegram-access-request.md](../persistencia/entidades/ENT-013-telegram-access-request.md), [guia-usuario.md](../guia-usuario.md), [FLUX-011 — aprovação de acesso](../fluxos/FLUX-011-aprovacao-acesso-telegram.md) |
| 4. Cadastro de arquivos, scripts e ações | [ARQ-004-registro-execucao-scripts.md](../arquitetura/ARQ-004-registro-execucao-scripts.md) | [ENT-015-script-registry.md](../persistencia/entidades/ENT-015-script-registry.md), [ENT-016-action-catalog.md](../persistencia/entidades/ENT-016-action-catalog.md), [INTF-TERM-001-sabiactl.md](../interfaces/terminal/INTF-TERM-001-sabiactl.md), [FLUX-012 — validação/publicação](../fluxos/FLUX-012-validacao-publicacao-script.md) |
| 5. Comando interno e navegação | [CONTR-001-comando-interno.md](../contratos/CONTR-001-comando-interno.md) | [FLUX-001-comando-telegram.md](../fluxos/FLUX-001-comando-telegram.md), [ARQ-003-core-command-router.md](../arquitetura/ARQ-003-core-command-router.md) |
| 6. Telegram: recebimento, entrega e recuperação | [FLUX-001-comando-telegram.md](../fluxos/FLUX-001-comando-telegram.md) | [REF-004-operacao-telegram.md](REF-004-operacao-telegram.md), [ENT-002-inbound-update.md](../persistencia/entidades/ENT-002-inbound-update.md) |
| 7. Jobs e processadores | [CONTR-003-job.md](../contratos/CONTR-003-job.md) | [CONTR-005-processador-assincrono.md](../contratos/CONTR-005-processador-assincrono.md), [FLUX-002-job-assincrono.md](../fluxos/FLUX-002-job-assincrono.md) |
| 8. Mídia: contratos e limites | [CONTR-009-ingestao-midia-fracionada.md](../contratos/CONTR-009-ingestao-midia-fracionada.md) | [CONTR-008-ingestao-midia-messagepack.md](../contratos/CONTR-008-ingestao-midia-messagepack.md), [ENT-008-media.md](../persistencia/entidades/ENT-008-media.md) |
| 9. Scheduler, alertas e assinaturas de eventos | [FLUX-003-monitoramento-agendado.md](../fluxos/FLUX-003-monitoramento-agendado.md) | [ARQ-009-scheduler-alertas.md](../arquitetura/ARQ-009-scheduler-alertas.md), [REF-006-gate-mvp.md](REF-006-gate-mvp.md), [CONTR-012 — assinaturas por eventos](../contratos/CONTR-012-assinaturas-eventos.md) |
| 10. Persistência e consistência | [readme.md](../persistencia/readme.md) | [PERS-001-relacionamentos.md](../persistencia/PERS-001-relacionamentos.md), [concorrencia.md](../persistencia/concorrencia.md) |
| 11. Configuração e operação local | [REF-001-modelo-configuracao.md](REF-001-modelo-configuracao.md) | [INTF-TERM-001-sabiactl.md](../interfaces/terminal/INTF-TERM-001-sabiactl.md), [GUIA-001-preparar-instalacao.md](../guias/GUIA-001-preparar-instalacao.md), [CONTR-011 — API administrativa UDS](../contratos/CONTR-011-api-administrativa-uds.md) |
| 12. Testes e critérios mínimos de aceite | [REF-003-requisitos-testes.md](REF-003-requisitos-testes.md) | [README.md](../testes/telegram/README.md) |
| 13. Pendências e conflitos que não podem ser inferidos | [REF-006-gate-mvp.md](REF-006-gate-mvp.md) | [pendencias.md](../persistencia/PERS-002-pendencias-modelo-fisico.md) |
| 14. Rastreabilidade compacta | [README.md](../README.md) | [README.md](README.md) |

## Divergências que precisam de decisão

- O grupo `abioma` de contratos antigos diverge do grupo proposto `absabia` para UDS do Sabiá. [Segurança](../arquitetura/ARQ-006-seguranca-autorizacao.md) documenta as duas referências.
- Regras de administração dinâmica de usuários, scripts, menus e ações ainda divergem de partes da configuração YAML estática. O [modelo de configuração](REF-001-modelo-configuracao.md) continua em `refinement`.
- Os requisitos de publicação e assinatura por tópicos foram **absorvidos** em [CONTR-012](../contratos/CONTR-012-assinaturas-eventos.md), mas continuam sem operações, payloads, autorização e persistência aprovados. Não confundir assinaturas com a aprovação de contatos do Telegram.
- O fluxo de upload, validação, isolamento e ativação foi **absorvido** em [FLUX-012](../fluxos/FLUX-012-validacao-publicacao-script.md), permanecendo em `refinement`. Não considerar a sintaxe `bash -n` verificação suficiente de segurança.
- O gate de especificação continua [bloqueado](REF-006-gate-mvp.md); não preencher lacunas usando comportamento inferido.

## Resultado da absorção documental

As **14 seções** da origem foram mapeadas por assunto na tabela acima, sem criar fontes concorrentes para os contratos anteriores. As lacunas específicas foram absorvidas em [CONTR-011](../contratos/CONTR-011-api-administrativa-uds.md), [CONTR-012](../contratos/CONTR-012-assinaturas-eventos.md), [FLUX-011](../fluxos/FLUX-011-aprovacao-acesso-telegram.md) e [FLUX-012](../fluxos/FLUX-012-validacao-publicacao-script.md).

**O que foi preservado como pendência, não aprovado:** regras de grupo Linux `abioma`/`absabia`, autoridades YAML/SQL, formato UDS administrativo, migrações e relações SQL, comportamento de alertas, retenção, publicação/assinatura de eventos e testes de scripts. O gate [REF-006](REF-006-gate-mvp.md) continua bloqueado. A migração integral do **conteúdo documental** não significa conclusão técnica do projeto.

## Referências históricas do consolidado

O documento original se apresenta como síntese de `docsLegado.md` com 279 documentos e artefatos. Esta quantidade é **uma declaração da origem**, não uma contagem verificada neste lote. Os caminhos abaixo são identificadores históricos citados na síntese; a existência individual de cada arquivo não foi comprovada.

| Assunto consolidado | Principais origens históricas |
|---|---|
| Propósito, escopo e exclusões | `docs/01-Project_Charter/charter/README.md`, `docs/01-Project_Charter/scope/README.md`, `docs/especificacao/requisitos/mvp.md` |
| Arquitetura, bots, segurança | `docs/decisoes/DEC-002-clientes-telegram-isolados.md`, `docs/decisoes/DEC-008-menor-privilegio-autorizacao.md`, `docs/seguranca-acessos.md` |
| Contatos e solicitações | `docs/01-Project_Charter/scope/flows/FLW-006-start-registro-de-contato-e-apresentacao-do-menu.md`, `docs/solicitacoes-acesso.md` |
| Administração e validação de scripts | `docs/guia-usuario.md`, `docs/contratos/CONTR-002-execucao-script.md`, `docs/especificacao/configuracao/modelo-configuracao.md` |
| Comandos e ações | `docs/contratos/CONTR-001-comando-interno.md`, `docs/01-Project_Charter/scope/flows/FLW-007-cadastro-administrativo-e-execucao-de-acao-persistente.md` |
| Telegram e filas | `docs/arquitetura/ARQ-008-adaptador-telegram.md`, `docs/telegram-operacao-2026.md`, `docs/especificacao/persistencia/claim-concorrente-filas.md` |
| Jobs e shutdown | `docs/contratos/CONTR-003-job.md`, `docs/contratos/CONTR-005-processador-assincrono.md`, `docs/decisoes/DEC-010-encerramento-jobs.md` |
| Mídia | `docs/contratos/CONTR-008-ingestao-midia-messagepack.md`, `docs/contratos/CONTR-009-ingestao-midia-fracionada.md`, `docs/contratos/CONTR-007-transmissao-midia.md` |
| Scheduler e assinatura de eventos | `docs/fluxos/FLUX-003-monitoramento-agendado.md`, `docs/charter/objectives/OBJ-009-assinatura-e-entrega-de-eventos-telegram.md` |
| Modelo relacional e bloqueios | `docs/especificacao/persistencia/tabelas-estado-operacional.md`, `docs/especificacao/persistencia/relacionamentos-estado-operacional.md`, `docs/pendencias.md` |

## Fonte preservada

[Regras consolidadas — origem legada](../regras-consolidadas-sabia.md), com 14 seções, badges e versão histórica `0.1`. A origem permanece disponível, mesmo após absorção, para auditoria editorial.
