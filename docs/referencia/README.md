# Referências técnicas do Sabiá

Índice dos documentos de configuração operacional, escopo do MVP e requisitos de testes. Os documentos representam **especificações revisadas**, não implementação comprovada.

| Documento canônico | Documento histórico | Estado de origem | Fonte preservada |
|---|---|---|---|
| [REF-001 — Modelo de configuração](REF-001-modelo-configuracao.md) | `CFG-0001` | `refinement` | [Original](../especificacao/configuracao/modelo-configuracao.md) |
| [REF-002 — Escopo do primeiro MVP](REF-002-escopo-primeiro-mvp.md) | `REQ-0001` | `refined` | [Original](../especificacao/requisitos/mvp.md) |
| [REF-003 — Requisitos de testes](REF-003-requisitos-testes.md) | `REQ-0002` | `refined` | [Original](../especificacao/requisitos/testes.md) |
| [REF-004 — Regras operacionais do Telegram](REF-004-operacao-telegram.md) | Operação Telegram (21/10/2026) | Histórico; não revalidado nesta revisão | [Original](../telegram-operacao-2026.md) |
| [REF-005 — Rastreabilidade das regras](REF-005-rastreabilidade-regras.md) | Consolidação histórica | Mapa revisado; origem migrada parcialmente | [Original legado](../regras-consolidadas-sabia.md) |
| [REF-006 — Gate do MVP](REF-006-gate-mvp.md) | Pendências documentais | `BLOCKED` histórico; não resolvido pela revisão | [Original](../pendencias.md) |

## Pendências documentais

- O schema YAML de conexão PostgreSQL, os caminhos dos sockets de mídia, a configuração de `schedules`, `logging` e comandos habilitados ainda requerem validação.
- A origem do modelo de configuração mantém valores históricos de grupo Linux `abioma`, diferentes da proposta posterior `absabia`.
- A emenda do limite de mídia usa o mesmo valor numérico antes e depois (`50000000` bytes); não presumir uma redução efetiva sem nova decisão.
- A cobertura de testes é um requisito, não prova de testes executados. Consulte os [fluxos](../fluxos/README.md) e os [contratos](../contratos/README.md) relacionados.

Os originais permanecem acessíveis; esta migração não executou scripts, testes nem alterações de configuração.
