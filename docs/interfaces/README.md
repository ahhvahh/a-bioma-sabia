# Interfaces do Sabiá

Índice dos canais e interfaces planejados. **Nenhum cliente administrativo ou servidor UDS executável foi verificado nesta branch.**

| Assunto | Documento |
|---|---|
| Inventário de canais, serviços e limites de transporte | [ARQ-011 — Inventário de canais](../arquitetura/ARQ-011-inventario-canais-servicos.md) |
| Terminal local administrativo `sabiactl` | [INTF-TERM-001 — Terminal administrativo](terminal/INTF-TERM-001-sabiactl.md) |
| Comandos e navegação humana, descrição conceitual | [Guia do usuário](../guia-usuario.md) |
| Adaptação de mensagens e callbacks do Telegram | [ARQ-008 — Adaptador](../arquitetura/ARQ-008-adaptador-telegram.md) |
| Mídia local, contratos de entrada | [CONTR-008](../contratos/CONTR-008-ingestao-midia-messagepack.md) e [CONTR-009](../contratos/CONTR-009-ingestao-midia-fracionada.md) |
| Fluxos de execução e retorno | [Índice de fluxos](../fluxos/README.md) |

A interface administrativa UDS não possui ainda um contrato técnico fechado. Não extrapolar os formatos MessagePack de mídia para consultas, administração, autorização ou publicação de ações.
