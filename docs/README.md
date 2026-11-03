# Sabiá — documentação técnica

Documentação do projeto **A-Bioma Sabiá**. Os documentos descrevem a solução planejada; a implementação não está disponível nesta branch.

## Guias e referências

| Assunto | Documento |
|---|---|
| Instalação e implantação | [Instalação](instalacao.md) |
| Criar bot Telegram | [Criar bot no Telegram](criar-bot-telegram.md) |
| Interações do operador e do usuário | [Guia do usuário](guia-usuario.md) |
| Arquitetura | [Arquitetura](arquitetura.md) |
| Banco de dados — visão geral | [Persistência](persistencia.md) |
| Interfaces, sockets e serviços | [Interfaces e serviços](interfaces-servicos.md) |
| Contratos técnicos | [Referência de contratos](referencia-contratos.md) |
| Segurança e grupos Linux | [Segurança e acessos](seguranca-acessos.md) |
| Regras consolidadas | [Regras consolidadas](regras-consolidadas-sabia.md) |
| Questões e bloqueios | [Pendências](pendencias.md) |

## Tabelas e modelo PostgreSQL

Os documentos de [`/docs/presistencia`](presistencia/README.md) foram recuperados da branch `develop`. **Os estados de refinamento de cada documento foram mantidos**; propostas em `refinement` não estão automaticamente aprovadas.

| Documento | Descrição |
|---|---|
| [Proposta de tabelas — PST-0001](presistencia/tabelas-estado-operacional.md) | `tabelas-estado-operacional.md` |
| [Relacionamentos — PST-0002](presistencia/relacionamentos-estado-operacional.md) | `relacionamentos-estado-operacional.md` |
| [Concorrência nas filas — PST-0003](presistencia/claim-concorrente-filas.md) | `claim-concorrente-filas.md` |
| [transport_cursor — PST-0101](presistencia/transport-cursor.md) | `transport-cursor.md` |
| [inbound_update — PST-0102](presistencia/inbound-update.md) | `inbound-update.md` |
| [request — PST-0103](presistencia/request.md) | `request.md` |
| [job — PST-0104](presistencia/job.md) | `job.md` |
| [job_state_history — PST-0105](presistencia/job-state-history.md) | `job-state-history.md` |
| [outbound_message — PST-0106](presistencia/outbound-message.md) | `outbound-message.md` |
| [alert_state — PST-0107](presistencia/alert-state.md) | `alert-state.md` |
| [media — PST-0108](presistencia/media.md) | `media.md` |
| [media_chunk — PST-0109](presistencia/media-chunk.md) | `media-chunk.md` |
| [media_transmission — PST-0110](presistencia/media-transmission.md) | `media-transmission.md` |
| [schema_version — PST-0111](presistencia/schema-version.md) | `schema-version.md` |

**Referência:** [índice das entidades persistentes](presistencia/README.md). Documentos e decisões anteriores ainda disponíveis no histórico de `develop`.
