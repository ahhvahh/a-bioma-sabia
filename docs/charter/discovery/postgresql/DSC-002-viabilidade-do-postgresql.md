# DSC-002 — Viabilidade da persistência PostgreSQL

![Tipo: DSC](../../assets/documentation-style/badges/badge-type-dsc.svg) ![Status: refinement](../../assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](../../assets/documentation-style/badges/badge-version-v1.svg)

## Questão e contexto

A integração com este serviço/ambiente é necessária ao escopo previsto.

## Evidência documental conhecida

Uma instância PostgreSQL compatível é obrigatória. A primeira versão do Sabiá utilizará **PostgreSQL 17**, com atualização para **PostgreSQL 18** em evolução futura e avaliação específica; conectividade e recuperação continuam dependentes de validação no ambiente.

## Decisão de versão inicial

- **Versão principal inicial:** PostgreSQL 17, usando a versão secundária corrigida disponível no repositório confiável escolhido para Debian 13 (Trixie). Não congelar um número de correção sem necessidade.
- **Evolução posterior:** PostgreSQL 18, somente após planejamento, backup, teste de migração, compatibilidade de aplicações e janela de manutenção. A escolha não autoriza atualização automática nem exige PostgreSQL 18 para o primeiro MVP.
- **Restrição preservada:** PostgreSQL permanece o banco operacional oficial, sem fallback normativo para SQLite. Escolhas de driver, tabelas e estratégias de atualização pertencem ao Scope/Implementation.

## Evidências oficiais consultadas em 01/11/2026

- Debian 13 (Trixie): o metapacote `postgresql` aponta para a série 17 — https://packages.debian.org/trixie/postgresql.
- Debian: o pacote `postgresql-17` integra o Trixie — https://packages.debian.org/trixie/postgresql-17.
- Política de versões e suporte PostgreSQL — https://www.postgresql.org/support/versioning/.
- Atualização de versão principal (`pg_upgrade` ou exportação/restauração) — https://www.postgresql.org/docs/17/upgrading.html.

## Plano de validação do ambiente

1. Confirmar Debian, pacote instalado e resultado de `SHOW server_version;` no servidor previsto, sem divulgar credenciais.
2. Testar conexão, autenticação e permissões mínimas do usuário de serviço do Sabiá.
3. Demonstrar gravação, leitura e reinicialização do banco com recuperação do estado necessário; registrar o resultado reproduzível.
4. Executar backup e teste de restauração antes da implantação operacional.
5. Para a evolução a PostgreSQL 18, testar migração em cópia dos dados, extensões e aplicação; planejar janela, reversão e verificação antes de aprovar a mudança.

Nenhuma instalação, conexão, migração ou restauração foi executada nesta rodada.

## Lacunas de verificação

- Confirmar a instalação real da versão 17, conectividade, credenciais, permissões, capacidade e testes reproduzíveis.
- Registrar evidência de backup, restauração e recuperação antes de promover a conclusão.
- Não interpretar a compatibilidade documental como validação ambiental ou aprovação de PostgreSQL 18.

## Conclusão

**Viabilidade documental da versão 17 confirmada; implantação ainda inconclusiva — `refinement`.** O pacote existe no Debian 13 e a decisão de versão foi registrada, mas falta verificar o ambiente real. O gate Charter → Scope permanece bloqueado pelas evidências pendentes.

## Dependências

[CON-005](../../conditions/CON-005-postgresql-e-a-persistencia-oficial-do-estado-operacional-se.md), [ASM-004](../../conditions/ASM-004-uma-instancia-postgresql-compativel-estara-disponivel-ao-ser.md), [OBJ-005](../../objectives/OBJ-005-preservar-estado-operacional.md); especificação de instalação, autenticação e migração deve seguir para Scope/Implementation, sem alterar essas etapas nesta rodada.
