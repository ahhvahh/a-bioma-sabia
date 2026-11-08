<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# REF-007 — Inventário de origens documentais consolidadas

**Escopo:** rodada de organização da branch `devDocumentacao`. Este inventário registra **14 fontes antigas ainda preservadas**, com documentos canônicos efetivamente existentes. Não altera a aprovação, a implementação nem a vigência técnica de nenhum requisito.

## Origens e fontes canônicas

| Origem preservada | Documento(s) canônico(s) | Estado da origem |
|---|---|---|
| [`docs/criar-bot-telegram.md`](../criar-bot-telegram.md) | [`docs/guias/GUIA-002-criar-bot-telegram.md`](../guias/GUIA-002-criar-bot-telegram.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/especificacao/configuracao/modelo-configuracao.md`](../especificacao/configuracao/modelo-configuracao.md) | [`docs/referencia/REF-001-modelo-configuracao.md`](REF-001-modelo-configuracao.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/especificacao/requisitos/mvp.md`](../especificacao/requisitos/mvp.md) | [`docs/referencia/REF-002-escopo-primeiro-mvp.md`](REF-002-escopo-primeiro-mvp.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/especificacao/requisitos/testes.md`](../especificacao/requisitos/testes.md) | [`docs/referencia/REF-003-requisitos-testes.md`](REF-003-requisitos-testes.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/instalacao.md`](../instalacao.md) | [`docs/guias/GUIA-001-preparar-instalacao.md`](../guias/GUIA-001-preparar-instalacao.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/interfaces-servicos.md`](../interfaces-servicos.md) | [`docs/arquitetura/ARQ-011-inventario-canais-servicos.md`](../arquitetura/ARQ-011-inventario-canais-servicos.md), [`docs/interfaces/terminal/INTF-TERM-001-sabiactl.md`](../interfaces/terminal/INTF-TERM-001-sabiactl.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/pendencias.md`](../pendencias.md) | [`docs/referencia/REF-006-gate-mvp.md`](REF-006-gate-mvp.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/persistencia/pendencias.md`](../persistencia/pendencias.md) | [`docs/persistencia/PERS-002-pendencias-modelo-fisico.md`](../persistencia/PERS-002-pendencias-modelo-fisico.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/persistencia/relacionamentos.md`](../persistencia/relacionamentos.md) | [`docs/persistencia/PERS-001-relacionamentos.md`](../persistencia/PERS-001-relacionamentos.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/regras-consolidadas-sabia.md`](../regras-consolidadas-sabia.md) | [`docs/referencia/REF-005-rastreabilidade-regras.md`](REF-005-rastreabilidade-regras.md), [`docs/contratos/CONTR-011-api-administrativa-uds.md`](../contratos/CONTR-011-api-administrativa-uds.md), [`docs/contratos/CONTR-012-assinaturas-eventos.md`](../contratos/CONTR-012-assinaturas-eventos.md), [`docs/fluxos/FLUX-011-aprovacao-acesso-telegram.md`](../fluxos/FLUX-011-aprovacao-acesso-telegram.md), [`docs/fluxos/FLUX-012-validacao-publicacao-script.md`](../fluxos/FLUX-012-validacao-publicacao-script.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/seguranca-acessos.md`](../seguranca-acessos.md) | [`docs/arquitetura/ARQ-006-seguranca-autorizacao.md`](../arquitetura/ARQ-006-seguranca-autorizacao.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/telegram-operacao-2026.md`](../telegram-operacao-2026.md) | [`docs/referencia/REF-004-operacao-telegram.md`](REF-004-operacao-telegram.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/arquitetura.md`](../arquitetura.md) | [`docs/arquitetura/ARQ-011-inventario-canais-servicos.md`](../arquitetura/ARQ-011-inventario-canais-servicos.md) | `OBSOLETO` · `MIGRADO` |
| [`docs/referencia-contratos.md`](../referencia-contratos.md) | [`docs/contratos/README.md`](../contratos/README.md) | `OBSOLETO` · `MIGRADO` |

Os estados `OBSOLETO` + `MIGRADO` identificam arquivos **documentalmente substituídos**, candidatos a uma futura limpeza **somente após autorização específica para exclusão**. O arquivo físico permanece acessível na branch, além do histórico Git. Não tratar essa classificação como prova de software executável.

## Critérios desta consolidação

- As 12 primeiras fontes já continham marcação `MIGRADO` e links para destinos canônicos. Suas seções e os destinos existentes foram reexaminados; as fontes foram classificadas `OBSOLETO`, sem excluir nem reescrever suas especificações técnicas.
- O antigo `docs/arquitetura.md` tinha dois diagramas Mermaid de topologia e sequência administrativa. Ambos passaram a constar no [ARQ-011](../arquitetura/ARQ-011-inventario-canais-servicos.md), com a distinção entre API UDS do Sabiá e UDS PostgreSQL; o conteúdo continua **proposto**.
- O antigo `docs/referencia-contratos.md` apenas apontava para o [índice canônico dos contratos](../contratos/README.md), que já preserva os links históricos `CTR-0001` a `CTR-0010`.
- Os índices de navegação do repositório apontam diretamente para os documentos canônicos. As fontes históricas permanecem relacionadas aqui para permitir auditoria e revisão posterior.

## Pendências que a organização não resolve

- O gate do MVP permanece documentado como `BLOCKED` em [REF-006](REF-006-gate-mvp.md).
- Os contratos antigos de mídia registram o grupo Linux `abioma`, enquanto a proposta posterior utiliza `absabia`. Não inferir uma política implantada: consulte [ARQ-006](../arquitetura/ARQ-006-seguranca-autorizacao.md).
- A experiência visual completa de solicitações de acesso não está coberta por `docs/solicitacoes-acesso.md`, que está ausente; o [terminal proposto](../interfaces/terminal/INTF-TERM-001-sabiactl.md) e o [fluxo](../fluxos/FLUX-011-aprovacao-acesso-telegram.md) ainda precisam de contratos executáveis.

## Próximo tratamento

Após decidir a limpeza histórica, confirmar novamente os links que apontam para cada origem, substituir referências ativas por destinos canônicos, converter links de rastreabilidade em permalinks para o commit anterior à remoção e, **somente com autorização**, excluir o lote. Esta rodada preserva os 14 arquivos.
