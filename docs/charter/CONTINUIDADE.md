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
| `discovery/postgresql/DSC-002-viabilidade-do-postgresql.md` | `refinement` | D-003 |
| `discovery/telegram-bot-api/README.md` | `refinement` | D-001 |
| `discovery/telegram-bot-api/DSC-001-viabilidade-da-telegram-bot-api.md` | `refinement` | D-002 |
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

Todos os documentos foram publicados e verificados na Fase B; os itens em `refinement` ainda exigem revisões independentes. Os SVGs foram recuperados da norma no SHA registrado.

## Pontos a revisar e questões abertas

| Demanda | Questão | Criticidade | Estado | Depende de | Próxima ação |
|---|---|---|---|---|---|
| D-001 | Migrar Charter, índices, badges e refs canônicas | P0 | concluido | Backup Fase A — confirmado | Commit `3e6f0e4c` verificado |
| D-002 | DSC-001 Telegram | P0 | em_analise | D-001 (concluído) | Realizar ensaio ambiental com perfil inicial; incorporar evidências e revisar se necessário |
| D-003 | DSC-002 PostgreSQL | P0 | em_analise | D-001 (concluído) | PostgreSQL 17 escolhido; verificar instalação, conexão, backup e recuperação |
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
- 01/11/2026: Fase B aprovada e publicada no commit [3e6f0e4c](https://github.com/ahhvahh/a-bioma-sabia/commit/3e6f0e4c0b8b7982b53d69faa545046dc8e331b1), 59 arquivos canônicos, sem exclusões.
- 01/11/2026: para D-002, o usuário escolheu **valores iniciais padrão e ajustáveis posteriormente**. Os números recomendados e o plano de testes constam em DSC-001 (commit [c768aa47](https://github.com/ahhvahh/a-bioma-sabia/commit/c768aa47ca432941d78249534c4f95ff45f33cef)), sujeitos à validação ambiental.
- 01/11/2026: para D-003, o usuário decidiu usar **PostgreSQL 17 na primeira versão e avaliar a evolução para 18 posteriormente**. A decisão é documentada no DSC-002; nenhum teste de ambiente foi executado.

## Progresso e próxima ação

- Backup: A_BACKUP_COMPLETO, verificado.
- Migração: B_MIGRACAO_COMPLETA; commit `3e6f0e4c` verificado.
- Limpeza: C_LIMPEZA_NAO_ELEGIVEL; originais intactos.
- Universo: 28 fontes de Charter e decisões relacionadas, 31 fontes de Scope apenas como evidência da etapa posterior.
- Próxima demanda: D-003, verificar o PostgreSQL 17 no ambiente previsto; D-002 mantém ensaios Telegram pendentes.

## Histórico de rodadas e reaberturas

| Rodada | Resultado | Evidência |
|---|---|---|
| 2026-11-01, Fase A | Backup concluído | 638a8542 |
| 2026-11-01, Fase B | Migração aprovada e publicada | `3e6f0e4c`; D-001 concluída |

## Identificação, baseline e autorização

- Alvo: https://github.com/ahhvahh/a-bioma-sabia
- Branch: develop
- Baseline analisada: `c768aa47ca432941d78249534c4f95ff45f33cef`
- Norma somente leitura: `20b49b182ea529f8b809e8e009132ab0fd5fe267`
- Skill Charter: 1.5.0
- Autorização da Fase B: diff aprovado pelo usuário e commit `3e6f0e4c` verificado.
- Autorização do diff D-002: confirmada pelo usuário; publicado e verificado no commit `c768aa47`.
- Autorização da D-003: a decisão de versão foi expressa pelo usuário; a publicação deste diff exige confirmação específica e verificação do commit remoto.

## Gate Charter → Scope

`BLOCKED`: migração concluída, mas DSCs obrigatórios permanecem inconclusivos no ambiente e documentos necessários ainda estão em `refinement`. A adoção de padrões não comprova viabilidade operacional.
