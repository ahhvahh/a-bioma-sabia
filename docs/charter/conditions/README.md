# Condições do Sabiá — restrições e premissas

![Tipo: GUIDE](../assets/documentation-style/badges/badge-type-guide.svg) ![Status: refinement](../assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](../assets/documentation-style/badges/badge-version-v1.svg)

## Restrições

- [CON-001 — Serviço Linux e binário gerenciado pelo systemd](CON-001-o-servico-opera-em-linux-e-o-executavel-principal-e-distribu.md)
- [CON-002 — Telegram por long polling](CON-002-o-primeiro-canal-telegram-recebe-updates-por-long-polling-e-.md)
- [CON-003 — Processo sem root](CON-003-o-processo-principal-nao-executa-como-root-e-utiliza-identid.md)
- [CON-004 — Operações cadastradas](CON-004-operacoes-remotas-somente-podem-resolver-identificadores-pre.md)
- [CON-005 — Persistência PostgreSQL](CON-005-postgresql-e-a-persistencia-oficial-do-estado-operacional-se.md)
- [CON-006 — Isolamento por cliente Telegram](CON-006-cada-cliente-telegram-mantem-identidade-autorizacao-e-catalo.md)
- [CON-007 — Autorização Linux das interfaces locais](CON-007-interfaces-locais-compartilhadas-com-aplicacoes-sao-limitada.md)
- [CON-008 — Contatos e catálogo persistidos](CON-008-contatos-e-catalogo-de-acoes-sao-persistidos-no-postgresql-o.md)
- [CON-009 — Administração local de executáveis](CON-009-cadastro-ou-alteracao-de-script-processador-associado-a-uma-.md)
- [CON-010 — Limites de ingestão de mídia](CON-010-limites-de-ingestao-de-midia.md)

## Premissas

- [ASM-001 — Conectividade de saída para Telegram](ASM-001-a-instalacao-possui-conectividade-de-saida-suficiente-para-a.md)
- [ASM-002 — Credenciais válidas e protegidas](ASM-002-tokens-e-demais-credenciais-validas-serao-fornecidos-por-mec.md)
- [ASM-003 — Ações previamente cadastradas](ASM-003-operacoes-disponiveis-aos-usuarios-serao-cadastradas-previam.md)
- [ASM-004 — PostgreSQL disponível](ASM-004-uma-instancia-postgresql-compativel-estara-disponivel-ao-ser.md)
- [ASM-005 — Identidades Linux autorizadas](ASM-005-aplicacoes-locais-que-utilizem-interfaces-compartilhadas-ser.md)
