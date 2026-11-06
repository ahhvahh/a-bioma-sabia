<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# REF-005 — Mapa de rastreabilidade das regras do Sabiá

**Objetivo:** localizar as fontes canônicas dos 14 assuntos do documento consolidado histórico, sem duplicar definições de contratos, políticas e esquemas.

**Situação:** índice documental verificado na branch `devDocumentacao`; **não equivale à comprovação de implementação**. O documento original continua **legado com absorção parcial**: nem todas as regras administrativas e de assinaturas por eventos têm contrato final aprovado.

## Assuntos e fontes canônicas

| Seção histórica | Fonte principal | Referências relacionadas |
|---|---|---|
| 1. Propósito e limites | [ARQ-001-contexto-sabia.md](../arquitetura/ARQ-001-contexto-sabia.md) | [REF-002-escopo-primeiro-mvp.md](REF-002-escopo-primeiro-mvp.md), [DEC-003-core-independente-telegram.md](../decisoes/DEC-003-core-independente-telegram.md) |
| 2. Identidades, isolamento e autorização | [ARQ-006-seguranca-autorizacao.md](../arquitetura/ARQ-006-seguranca-autorizacao.md) | [DEC-008-menor-privilegio-autorizacao.md](../decisoes/DEC-008-menor-privilegio-autorizacao.md) |
| 3. Bots, contatos e solicitações de acesso | [ARQ-008-adaptador-telegram.md](../arquitetura/ARQ-008-adaptador-telegram.md) | [ENT-013-telegram-access-request.md](../persistencia/entidades/ENT-013-telegram-access-request.md), [guia-usuario.md](../guia-usuario.md) |
| 4. Cadastro de arquivos, scripts e ações | [ARQ-004-registro-execucao-scripts.md](../arquitetura/ARQ-004-registro-execucao-scripts.md) | [ENT-015-script-registry.md](../persistencia/entidades/ENT-015-script-registry.md), [ENT-016-action-catalog.md](../persistencia/entidades/ENT-016-action-catalog.md), [INTF-TERM-001-sabiactl.md](../interfaces/terminal/INTF-TERM-001-sabiactl.md) |
| 5. Comando interno e navegação | [CONTR-001-comando-interno.md](../contratos/CONTR-001-comando-interno.md) | [FLUX-001-comando-telegram.md](../fluxos/FLUX-001-comando-telegram.md), [ARQ-003-core-command-router.md](../arquitetura/ARQ-003-core-command-router.md) |
| 6. Telegram: recebimento, entrega e recuperação | [FLUX-001-comando-telegram.md](../fluxos/FLUX-001-comando-telegram.md) | [REF-004-operacao-telegram.md](REF-004-operacao-telegram.md), [ENT-002-inbound-update.md](../persistencia/entidades/ENT-002-inbound-update.md) |
| 7. Jobs e processadores | [CONTR-003-job.md](../contratos/CONTR-003-job.md) | [CONTR-005-processador-assincrono.md](../contratos/CONTR-005-processador-assincrono.md), [FLUX-002-job-assincrono.md](../fluxos/FLUX-002-job-assincrono.md) |
| 8. Mídia: contratos e limites | [CONTR-009-ingestao-midia-fracionada.md](../contratos/CONTR-009-ingestao-midia-fracionada.md) | [CONTR-008-ingestao-midia-messagepack.md](../contratos/CONTR-008-ingestao-midia-messagepack.md), [ENT-008-media.md](../persistencia/entidades/ENT-008-media.md) |
| 9. Scheduler, alertas e assinaturas de eventos | [FLUX-003-monitoramento-agendado.md](../fluxos/FLUX-003-monitoramento-agendado.md) | [ARQ-009-scheduler-alertas.md](../arquitetura/ARQ-009-scheduler-alertas.md), [REF-006-gate-mvp.md](REF-006-gate-mvp.md) |
| 10. Persistência e consistência | [readme.md](../persistencia/readme.md) | [PERS-001-relacionamentos.md](../persistencia/PERS-001-relacionamentos.md), [concorrencia.md](../persistencia/concorrencia.md) |
| 11. Configuração e operação local | [REF-001-modelo-configuracao.md](REF-001-modelo-configuracao.md) | [INTF-TERM-001-sabiactl.md](../interfaces/terminal/INTF-TERM-001-sabiactl.md), [GUIA-001-preparar-instalacao.md](../guias/GUIA-001-preparar-instalacao.md) |
| 12. Testes e critérios mínimos de aceite | [REF-003-requisitos-testes.md](REF-003-requisitos-testes.md) | [README.md](../testes/telegram/README.md) |
| 13. Pendências e conflitos que não podem ser inferidos | [REF-006-gate-mvp.md](REF-006-gate-mvp.md) | [pendencias.md](../persistencia/pendencias.md) |
| 14. Rastreabilidade compacta | [README.md](../README.md) | [README.md](README.md) |

## Divergências que precisam de decisão

- O grupo `abioma` de contratos antigos diverge do grupo proposto `absabia` para UDS do Sabiá. [Segurança](../arquitetura/ARQ-006-seguranca-autorizacao.md) documenta as duas referências.
- Regras de administração dinâmica de usuários, scripts, menus e ações ainda divergem de partes da configuração YAML estática. O [modelo de configuração](REF-001-modelo-configuracao.md) continua em `refinement`.
- Publicação e assinatura por tópicos de eventos, citadas na seção histórica 9, ainda carecem de contratos de publicação, cancelamento, autorização e persistência. Não confundir assinaturas com a aprovação de contatos do Telegram.
- O fluxo de upload, validação, isolamento e ativação de scripts permanece parcialmente proposto. Não considerar a sintaxe `bash -n` verificação suficiente de segurança.
- O gate de especificação continua [bloqueado](REF-006-gate-mvp.md); não preencher lacunas usando comportamento inferido.

## Rastreabilidade e limite de migração

Fonte original: [Regras consolidadas do Sabiá](../regras-consolidadas-sabia.md), com 14 seções e origens históricas. Este mapa **não declara migração integral** das 29 mil posições de conteúdo; o original permanece preservado, sem o marcador `MIGRADO`, até confronto integral dos pontos que ainda não têm contrato canônico.
