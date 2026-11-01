# Continuidade — Charter do Sabiá

![Tipo: GUIDE](assets/documentation-style/badges/badge-type-guide.svg) ![Status: refinement](assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](assets/documentation-style/badges/badge-version-v1.svg)

## Ideia geral do projeto

O projeto atende operadores, usuários Telegram autorizados e aplicações locais. O [Charter principal](README.md#propósito-do-projeto) apresenta o propósito e os limites.

## O que o projeto fará

| OBJ | Capacidade | Situação | Demanda |
|---|---|---|---|
| [OBJ-001](objectives/OBJ-001-acionar-operacoes-locais-cadastradas.md) | Acionar operações locais cadastradas | Revisão pendente | D-001 |
| [OBJ-002](objectives/OBJ-002-processamento-assincrono.md) | Processamento assíncrono | Revisão pendente | D-001 |
| [OBJ-003](objectives/OBJ-003-verificacoes-agendadas-e-alertas.md) | Monitoramento e alertas | Revisão pendente | D-001 |
| [OBJ-004](objectives/OBJ-004-ingestao-e-entrega-de-midia.md) | Ingestão e entrega de mídia | Revisão pendente | D-001 |
| [OBJ-005](objectives/OBJ-005-preservar-estado-operacional.md) | Persistência e recuperação | Revisão pendente | D-001 |
| [OBJ-006](objectives/OBJ-006-restringir-acesso-e-execucao.md) | Restringir acesso e execução | Revisão pendente | D-001 |
| [OBJ-007](objectives/OBJ-007-registrar-contatos-telegram.md) | Registrar contatos Telegram | Revisão pendente | D-001 |
| [OBJ-008](objectives/OBJ-008-administrar-catalogo-de-acoes.md) | Administrar catálogo de ações | Revisão pendente | D-001 |
| [OBJ-009](objectives/OBJ-009-assinatura-e-entrega-de-eventos-telegram.md) | Assinaturas de tópicos e notificações por evento | Revisão pendente | D-001 |

## O que o projeto não fará

| NOBJ | Exclusão | Situação | Demanda |
|---|---|---|---|
| NOBJ-001 | Não oferecer shell remoto arbitrário | Revisão pendente | D-001 |
| NOBJ-002 | Não expor webhook público no primeiro MVP | Revisão pendente | D-001 |
| NOBJ-003 | Não implementar processadores multimídia complexos no MVP | Revisão pendente | D-001 |
| NOBJ-004 | Não expor canais remotos adicionais no MVP | Revisão pendente | D-001 |
| NOBJ-005 | Não conceder acesso direto a banco e diretórios | Revisão pendente | D-001 |

[NOBJ-006](non-objectives/NOBJ-006-assinaturas-excluidas-cancelado.md): cancelado por decisão de 21/10/2026; a exclusão de assinaturas por evento deixou de valer.

## Mapa de documentos e cobertura

| Arquivo canônico | Status proposto | Demanda |
|---|---|---|
| `README.md` | `refinement` | D-001 |
| `conditions/ASM-001-a-instalacao-possui-conectividade-de-saida-suficiente-para-a.md` | `refinement` | D-001 |
| `conditions/ASM-002-tokens-e-demais-credenciais-validas-serao-fornecidos-por-mec.md` | `refinement` | D-001 |
| `conditions/ASM-003-operacoes-disponiveis-aos-usuarios-serao-cadastradas-previam.md` | `refinement` | D-001 |
| `conditions/ASM-004-uma-instancia-postgresql-compativel-estara-disponivel-ao-ser.md` | `refinement` | D-001 |
| `conditions/ASM-005-aplicacoes-locais-que-utilizem-interfaces-compartilhadas-ser.md` | `refinement` | D-001 |
| `conditions/CON-001-o-servico-opera-em-linux-e-o-executavel-principal-e-distribu.md` | `refinement` | D-001 |
| `conditions/CON-002-o-primeiro-canal-telegram-recebe-updates-por-long-polling-e-.md` | `refinement` | D-001 |
| `conditions/CON-003-o-processo-principal-nao-executa-como-root-e-utiliza-identid.md` | `refinement` | D-001 |
| `conditions/CON-004-operacoes-remotas-somente-podem-resolver-identificadores-pre.md` | `refinement` | D-001 |
| `conditions/CON-005-postgresql-e-a-persistencia-oficial-do-estado-operacional-se.md` | `refinement` | D-001 |
| `conditions/CON-006-cada-cliente-telegram-mantem-identidade-autorizacao-e-catalo.md` | `refinement` | D-001 |
| `conditions/CON-007-interfaces-locais-compartilhadas-com-aplicacoes-sao-limitada.md` | `refinement` | D-001 |
| `conditions/CON-008-contatos-e-catalogo-de-acoes-sao-persistidos-no-postgresql-o.md` | `refinement` | D-001 |
| `conditions/CON-009-cadastro-ou-alteracao-de-script-processador-associado-a-uma-.md` | `refinement` | D-001 |
| `conditions/CON-010-limites-de-ingestao-de-midia.md` | `refinement` | D-001 |
| `conditions/README.md` | `refinement` | D-001 |
| `discovery/README.md` | `refinement` | D-001 |
| `discovery/debian-systemd/README.md` | `refinement` | D-001 |
| `discovery/debian-systemd/DSC-003-viabilidade-do-servico-linux.md` | `refinement` | D-001 |
| `discovery/integracoes-locais/README.md` | `refinement` | D-001 |
| `discovery/integracoes-locais/DSC-004-viabilidade-de-integracoes-locais.md` | `refinement` | D-001 |
| `discovery/postgresql/README.md` | `refinement` | D-001 |
| `discovery/postgresql/DSC-002-viabilidade-do-postgresql.md` | `refinement` | D-001 |
| `discovery/telegram-bot-api/README.md` | `refinement` | D-001 |
| `discovery/telegram-bot-api/DSC-001-viabilidade-da-telegram-bot-api.md` | `refinement` | D-001 |
| `non-objectives/NOBJ-001-nao-oferecer-shell-remoto-arbitrario.md` | `refinement` | D-001 |
| `non-objectives/NOBJ-002-nao-expor-webhook-publico-no-mvp.md` | `refinement` | D-001 |
| `non-objectives/NOBJ-003-nao-criar-processadores-complexos-no-mvp.md` | `refinement` | D-001 |
| `non-objectives/NOBJ-004-nao-expor-canais-remotos-extras-no-mvp.md` | `refinement` | D-001 |
| `non-objectives/NOBJ-005-nao-dar-acesso-direto-ao-banco-ou-diretorios.md` | `refinement` | D-001 |
| `non-objectives/NOBJ-006-assinaturas-excluidas-cancelado.md` | `cancelled` | D-001 |
| `non-objectives/README.md` | `refinement` | D-001 |
| `objectives/OBJ-001-acionar-operacoes-locais-cadastradas.md` | `refinement` | D-001 |
| `objectives/OBJ-002-processamento-assincrono.md` | `refinement` | D-001 |
| `objectives/OBJ-003-verificacoes-agendadas-e-alertas.md` | `refinement` | D-001 |
| `objectives/OBJ-004-ingestao-e-entrega-de-midia.md` | `refinement` | D-001 |
| `objectives/OBJ-005-preservar-estado-operacional.md` | `refinement` | D-001 |
| `objectives/OBJ-006-restringir-acesso-e-execucao.md` | `refinement` | D-001 |
| `objectives/OBJ-007-registrar-contatos-telegram.md` | `refinement` | D-001 |
| `objectives/OBJ-008-administrar-catalogo-de-acoes.md` | `refinement` | D-001 |
| `objectives/OBJ-009-assinatura-e-entrega-de-eventos-telegram.md` | `refinement` | D-001 |
| `objectives/README.md` | `refinement` | D-001 |
| `open-questions/OPEN-001-autoridade-formal-de-aprovacao.md` | `refinement` | D-001 |
| `open-questions/README.md` | `refinement` | D-001 |
| `project-context/PCH-001-contexto-e-atores-do-sabia.md` | `refinement` | D-001 |
| `project-context/README.md` | `refinement` | D-001 |

Todos são documentos PROPOSTOS na prévia; os status só serão confirmados após commit e verificação. Os SVGs devem ser recuperados da norma em SHA fixado.

## Pontos a revisar e questões abertas

| Demanda | Questão | Criticidade | Estado | Depende de | Próxima ação |
|---|---|---|---|---|---|
| D-001 | Migrar Charter, índices, badges e refs canônicas | P0 | aguardando diff e aprovação | Backup Fase A — confirmado | Conferir conteúdo integral e commit autorizado |
| D-002 | DSC-001 Telegram | P0 | a_analisar | D-001 | Validar viabilidade e condições |
| D-003 | DSC-002 PostgreSQL | P0 | a_analisar | D-001 | Validar compatibilidade real |
| D-004 | DSC-003/004 ambiente Linux e integrações locais | P1 | a_analisar | D-001 | Validar ambiente e atores |
| D-005 | ACT-001..003 só no Scope legado | P0 | a_analisar | D-001 | Materializar cadastro canônico em PCH-001 |
| D-006 | NOBJ-001..005 somente em índice legado | P1 | a_analisar | D-001 | Desdobrar NOBJ e registrar consequências |
| D-007 | OBJ-009 ativo e OUT-009 legado não alinhado | P1 | bloqueado_outra_etapa | Migração Scope | Encaminhar revisão OUT-009 ao agente Scope |
| D-008 | OPEN-001 — autoridade formal | P2 | aguardando_decisao | nenhuma | Identificar papel futuro sem bloquear esta migração |
| D-009 | CON-010 — revisão de unidade e limite | P1 | a_analisar | D-001 | Usar 50.000.000 bytes como teto revisto |

## Decisões confirmadas

- 21/10/2026: eventos por assinatura incluídos (OBJ-009); NOBJ-006 cancelado; IN-013 periódico continua em backlog.
- 21/10/2026: CON-010 revisto para **50.000.000 bytes** como limite total fracionado.
- Fase A autorizada pelo usuário e publicada em commit [638a8542](https://github.com/ahhvahh/a-bioma-sabia/commit/638a8542a7195b18debdce8c2f3c711c6f8b5357), com 61 originais e manifesto.
- 01/11/2026: pedido de Fase B com índices, badges e análise de dependências. Não é autorização de conteúdo ainda não exibido.

## Progresso e próxima ação

- Backup: A_BACKUP_COMPLETO, verificado.
- Migração: B_MIGRACAO_PENDENTE; prévia documental preparada, sem commit.
- Limpeza: C_LIMPEZA_NAO_ELEGIVEL; originais intactos.
- Universo: 28 fontes de Charter e decisões relacionadas, 31 fontes de Scope apenas como evidência da etapa posterior.
- Próxima demanda: D-001, publicação de diff de migração após verificação editorial.

## Histórico de rodadas e reaberturas

| Rodada | Resultado | Evidência |
|---|---|---|
| 2026-11-01, Fase A | Backup concluído | 638a8542 |
| 2026-11-01, Fase B | Revisão e elaboração de prévia | D-001..009; publicação pendente |

## Identificação, baseline e autorização

- Alvo: https://github.com/ahhvahh/a-bioma-sabia
- Branch: develop
- Baseline analisada: `638a8542a7195b18debdce8c2f3c711c6f8b5357`
- Norma somente leitura: `20b49b182ea529f8b809e8e009132ab0fd5fe267`
- Skill Charter: 1.5.0
- Autorização de commit da Fase B: **pendente de diff completo revisável**.

## Gate Charter → Scope

`BLOCKED`: migração pendente, DSCs inconclusivos, documentos necessários ainda em `refinement`. Não promove-los por formatação.
