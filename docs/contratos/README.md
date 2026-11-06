# Contratos técnicos — Sabiá

Índice canônico das especificações técnicas conforme o A-Bioma Doc Standard. **Implementação:** não verificada nesta branch.

O prefixo `CONTR` identifica o destino revisado; `CTR` indica a fonte preservada. O estado de revisão documental não equivale a aprovação de contratos em `refinement`.

| Documento canônico | Identificador histórico | Estado na origem | Fonte histórica |
|---|---|---|---|
| [CONTR-001 — Comando interno](CONTR-001-comando-interno.md) | `CTR-0001` | `refined` | [Original](../especificacao/contratos/comando-interno.md) |
| [CONTR-002 — Execução de script](CONTR-002-execucao-script.md) | `CTR-0002` | `refined` | [Original](../especificacao/contratos/execucao-script.md) |
| [CONTR-003 — Job](CONTR-003-job.md) | `CTR-0003` | `refined` | [Original](../especificacao/contratos/job.md) |
| [CONTR-004 — Auditoria e logs](CONTR-004-auditoria-logs.md) | `CTR-0004` | `refinement` | [Original](../especificacao/contratos/auditoria-logs.md) |
| [CONTR-005 — Protocolo de processador assíncrono](CONTR-005-processador-assincrono.md) | `CTR-0005` | `refined` | [Original](../especificacao/contratos/processador-assincrono.md) |
| [CONTR-006 — Mídia persistida](CONTR-006-midia-persistida.md) | `CTR-0006` | `refined` | [Original](../especificacao/contratos/midia-persistida.md) |
| [CONTR-007 — Transmissão persistente de mídia](CONTR-007-transmissao-midia.md) | `CTR-0007` | `refined` | [Original](../especificacao/contratos/transmissao-midia.md) |
| [CONTR-008 — Ingestão de mídia por Unix socket e MessagePack](CONTR-008-ingestao-midia-messagepack.md) | `CTR-0008` | `refinement` | [Original](../especificacao/contratos/ingestao-midia-messagepack.md) |
| [CONTR-009 — Ingestão fracionada de mídia por Unix socket](CONTR-009-ingestao-midia-fracionada.md) | `CTR-0009` | `refinement` | [Original](../especificacao/contratos/ingestao-midia-fracionada.md) |
| [CONTR-010 — Origem e conversão dos dados persistentes](CONTR-010-origem-dados-persistencia.md) | `CTR-0010` | `refinement` | [Original](../especificacao/contratos/origem-dados-persistencia.md) |

## Pendências da revisão

- `CONTR-004`, `CONTR-008`, `CONTR-009` e `CONTR-010` permanecem em `refinement` e contêm decisões `BLOCKED`.
- Divergência histórica entre os grupos Linux `abioma` e `absabia` nos sockets de mídia; consultar [segurança](../seguranca-acessos.md).
- Em `CONTR-006`, a emenda informa redução de 50 MB para 50 MB, uma inconsistência textual. Permanece documentado o teto de `50000000` bytes para o canal fracionado.
- Em `CONTR-010`, o arquivo citado `docs/telegram-discovery.md` não existe nesta branch e não recebeu substituição presumida.

Os fluxos em `docs/especificacao/fluxos/` ainda podem referenciar os IDs históricos `CTR` e serão tratados no lote seguinte.
