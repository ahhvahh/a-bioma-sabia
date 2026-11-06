# Fluxos do Sabiá

Índice dos fluxos canônicos documentados conforme o **A-Bioma Doc Standard**. A implementação e a execução dos testes não foram verificadas nesta branch.

`FLUX` identifica o documento revisado; `FLW` é o ID histórico mantido na origem. `REVISADO` classifica a revisão documental, não a aprovação do fluxo.

| Documento canônico | ID original | Estado da especificação | Origem histórica |
|---|---|---|---|
| [FLUX-001 — Comando Telegram](FLUX-001-comando-telegram.md) | `FLW-0001` | `refinement` | [Original](../especificacao/fluxos/comando-telegram.md) |
| [FLUX-002 — Job assíncrono](FLUX-002-job-assincrono.md) | `FLW-0002` | `refinement` | [Original](../especificacao/fluxos/job-assincrono.md) |
| [FLUX-003 — Monitoramento agendado](FLUX-003-monitoramento-agendado.md) | `FLW-0003` | `refinement` | [Original](../especificacao/fluxos/monitoramento-agendado.md) |
| [FLUX-004 — Encerramento do serviço](FLUX-004-encerramento-servico.md) | `FLW-0004` | `refined` | [Original](../especificacao/fluxos/encerramento-servico.md) |
| [FLUX-005 — Processamento assíncrono por processador registrado](FLUX-005-processamento-assincrono.md) | `FLW-0005` | `refined` | [Original](../especificacao/fluxos/processamento-assincrono.md) |
| [FLUX-006 — Ingestão local de mídia](FLUX-006-ingestao-midia.md) | `FLW-0006` | `refinement` | [Original](../especificacao/fluxos/ingestao-midia.md) |
| [FLUX-007 — Ingestão fracionada de mídia](FLUX-007-ingestao-midia-fracionada.md) | `FLW-0007` | `refinement` | [Original](../especificacao/fluxos/ingestao-midia-fracionada.md) |
| [FLUX-008 — Limpeza de payloads de mídia transmitida](FLUX-008-limpeza-midia.md) | `FLW-0008` | `refined` | [Original](../especificacao/fluxos/limpeza-midia.md) |
| [FLUX-009 — Registro do estado operacional](FLUX-009-registro-estado-operacional.md) | `FLW-0009` | `refinement` | [Original](../especificacao/fluxos/registro-estado-operacional.md) |
| [FLUX-010 — Consumo do estado operacional](FLUX-010-consumo-estado-operacional.md) | `FLW-0010` | `refinement` | [Original](../especificacao/fluxos/consumo-estado-operacional.md) |

## Pendências conhecidas

- `FLUX-003`: correlação de alertas sem `request_id` e especificação do Scheduler ainda estão `BLOCKED`.
- `FLUX-007`: o `FLW-0007` histórico deixava o limite total indefinido; o contrato posterior [CONTR-009](../contratos/CONTR-009-ingestao-midia-fracionada.md) fixa `50000000` bytes (50 MB decimais). O documento continua em `refinement`.
- `FLUX-009`: depende do encerramento das pendências de [CONTR-010](../contratos/CONTR-010-origem-dados-persistencia.md) e das entidades afetadas.
- `FLUX-010`: a resposta a registros com domínio inválido após migração ainda requer decisão.
- Referências técnicas e grupos Linux conflitantes devem ser reconciliados com [Segurança e acessos](../seguranca-acessos.md).

Nenhum diagrama comprova implantação, execução real ou teste integrado. Os originais foram preservados sem exclusões.
