# Guias operacionais — A-Bioma Sabiá

Este índice reúne procedimentos orientados ao operador. **A instalação do Sabiá e a interface `sabiactl` não foram verificadas em execução.** Os passos documentados para esses componentes são preparação e planejamento, não instruções de produção testadas.

| Documento canônico | Origem | Uso |
|---|---|---|
| [GUIA-001 — Preparar a instalação](GUIA-001-preparar-instalacao.md) | [Instalação original](../instalacao.md) | Requisitos de Debian, systemd, identidade Linux e PostgreSQL; sem comandos executáveis |
| [GUIA-002 — Criar bot Telegram](GUIA-002-criar-bot-telegram.md) | [Guia BotFather original](../criar-bot-telegram.md) | Criar e proteger o token de um bot; a integração com Sabiá ainda é planejada |

O [Guia do usuário](../guia-usuario.md) mantém seu caminho estável e descreve a interação proposta por terminal e Telegram. A [segurança e autorização](../arquitetura/ARQ-006-seguranca-autorizacao.md) concentram as regras de isolamento e as divergências de grupos Linux.

## Pendências

- Implementação e validação do instalador, do serviço, das migrações e do `sabiactl`.
- Aprovação dos caminhos UDS, do modelo de credenciais PostgreSQL e das regras administrativas.
- Reconciliação entre `abioma` (legado) e `absabia` (proposto) antes da implantação.
