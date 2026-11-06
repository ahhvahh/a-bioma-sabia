<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# INTF-TERM-001 — Terminal administrativo `sabiactl`

**Plataforma:** terminal Linux local; interface textual navegável **planejada, não implementada/verificada**. Público: operador com conexão autorizada ao UDS do Sabiá.

## Objetivo

Permitir consulta e administração do serviço sem acesso SQL direto, preservando autorização por operação e confirmação de ações sensíveis. [Guia humano de uso](../../guia-usuario.md).

## Transporte e acesso

- Cliente local previsto: `sabiactl` usando [API administrativa UDS](../../arquitetura/ARQ-011-inventario-canais-servicos.md).
- Grupo `absabia` proposto para conexão. Operações privilegiadas exigem autorização adicional no serviço.
- **Não há caminho de socket, sintaxe CLI, flags, envelope, framing, serialização, códigos de saída, formato stdout/stderr ou implementação confirmada**. Não oferecer comandos de invocação como se fossem executáveis.
- `sabiactl` não acessa PostgreSQL nem arquivos privados diretamente. O Sabiá aplica validações, persistência e auditoria.

## Ações previstas e contratos

| Área / interação | Dados necessários | Contrato ou referência | Resultado esperado | Situação |
|---|---|---|---|---|
| Estado, conexões e serviços | Componente ou filtros de consulta | [Arquitetura dos canais](../../arquitetura/ARQ-011-inventario-canais-servicos.md) | Indicadores e contagens sem valores inventados | Operação UDS não definida |
| Testes de operação | Alvo e verificação permitida | [Requisitos de testes](../../referencia/REF-003-requisitos-testes.md) | Resultado, duração e falhas | Operação UDS não definida |
| Solicitações de acesso | Bot, pedido, ação e confirmação | [ENT-013](../../persistencia/entidades/ENT-013-telegram-access-request.md), [ENT-014](../../persistencia/entidades/ENT-014-telegram-authorization.md) | Lista paginada, decisão auditável | Sem contrato UDS/tela aprovado |
| Scripts e arquivos | Arquivo local, metadados, validação e identificação lógica | [ARQ-004](../../arquitetura/ARQ-004-registro-execucao-scripts.md), [ENT-015](../../persistencia/entidades/ENT-015-script-registry.md) | Validação antes da incorporação/ativação | Upload UDS sem contrato |
| Menus e ações | Bot, catálogo, operação vinculada e permissões | [ENT-016](../../persistencia/entidades/ENT-016-action-catalog.md), [fluxo de comandos](../../fluxos/FLUX-001-comando-telegram.md) | Publicação autorizada; catálogo filtrado | Persistência e autorização em revisão |
| Configurações | Chaves e valores permitidos | [REF-001](../../referencia/REF-001-modelo-configuracao.md) | Resultado sem credenciais ou segredos | Fonte de verdade YAML/SQL pendente |

As entidades vinculadas são **propostas físicas**, não contratos de API. O contrato de requisição/resposta do UDS administrativo ainda precisa ser criado.

## Navegação proposta

Para solicitações de acesso por bot, o consolidado histórico documenta: `↑`/`↓` para selecionar pedido, `→` para preparar aprovação, `←` para preparar rejeição e confirmação `Y` em ambas; `N` ou `Esc` cancela. O terminal deve exibir bot, nome e identificador numérico, sem tratar username como prova de identidade.

Para scripts, o fluxo previsto é **arquivo local → pré-validação → upload isolado → validação no serviço → confirmação → incorporação**. O upload **não é ativação nem publicação**. Ação de menu exige vínculo explícito com script validado e autorização por bot.

## Estados e falhas a especificar

- Inicial, lista vazia, carregando, autorizado, negado, validação falhou, conflito entre operadores, operação cancelada, timeout e serviço indisponível.
- Não inventar métricas, sucesso de upload ou texto definitivo de erros.
- Operações sensíveis devem confirmar a identidade do operador e registrar auditoria transacional; a aprovação não pode ser baseada só em pertencer ao grupo Linux.

## Referências e rastreabilidade

- [Inventário original de interfaces](../../interfaces-servicos.md).
- [Regras consolidadas históricas](../../regras-consolidadas-sabia.md), seções 3, 4 e 11 (absorção parcial).
- [Pendências do gate](../../referencia/REF-006-gate-mvp.md).
