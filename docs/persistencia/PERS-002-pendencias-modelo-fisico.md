<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# PERS-002 — Pendências do modelo físico PostgreSQL do Sabiá

**Tipo:** referência de decisões físicas pendentes do banco. **Estado:** `refinement` — nenhuma das decisões `DB-01` a `DB-12` está aprovada ou comprovadamente implementada nesta branch.

## Objetivo

Identificar as decisões que impedem fechar o DDL, o provisionamento ou a integridade do modelo PostgreSQL, sem inferir tabelas ou campos novos. O banco lógico `abioma_sabia`, o schema `sabia` e as roles citadas são **propostas documentais**, não objetos verificados.

## Inventário das decisões pendentes

| ID | Decisão pendente | Efeito na criação das tabelas |
|---|---|---|
| DB-01 | Versão PostgreSQL, host/cluster, diretório de UDS, `pg_hba.conf`, restrição de TCP e procedimento de backup | Bloqueia implantação segura |
| DB-02 | Mecanismo de migração e esquema de versionamento | `schema_version` poderá ser descartada |
| DB-03 | FK canônica entre `inbound_update` e `request` | Evitar duas colunas redundantes |
| DB-04 | Alertas do scheduler sem `request_id` | Corrigir contrato de `outbound_message` antes do DDL final |
| DB-05 | Relação `media_transmission.request_id` / `media.request_id` | Garantir coerência transacional |
| DB-06 | Retenção, purge, backups e recuperação de `raw_update`, históricos e binários | Restringir exclusões e dados pessoais |
| DB-07 | Precedência entre usuários/scripts/ações em YAML e PostgreSQL | Bloqueia desenho definitivo das cinco entidades administrativas propostas |
| DB-08 | Auditoria das decisões de acesso e ativação de scripts | Definir eventos/estrutura ou fonte auditável |
| DB-09 | Assinaturas por evento, tópicos, publicações e entregas | Entidades adicionais dependem de contratos funcionais ainda em refinamento |
| DB-10 | Arquivos administrativos e validação de scripts (binário, versão, armazenamento) | Definir se haverá `bytea` ou referência a arquivos privados |
| DB-11 | Contrato PostgreSQL em `/etc/sabia/sabia.yaml` e identidade de migração | Não inferir formato YAML nem credenciais |
| DB-12 | Validação das cinco tabelas administrativas e suas chaves | Proposta não equivale a modelo aprovado |

## Fontes canônicas para análise

| Decisões | Documento de referência | Limite da evidência |
|---|---|---|
| `DB-01`, `DB-11` | [Configuração PostgreSQL](configuracao-banco.md), [modelo YAML](../referencia/REF-001-modelo-configuracao.md) | Há exemplos de provisionamento, não configuração executada ou schema YAML aprovado |
| `DB-02` | [ENT-011 — schema_version](entidades/ENT-011-schema-version.md) | A entidade é condicional ao migrador escolhido |
| `DB-03`, `DB-04`, `DB-05` | [PERS-001 — Relacionamentos](PERS-001-relacionamentos.md), [ENT-006 — Mensagens](entidades/ENT-006-outbound-message.md), [ENT-010 — Transmissão](entidades/ENT-010-media-transmission.md) | Direção da FK, correlação de alerta e coerência de request requerem decisão |
| `DB-06` | [Concorrência e histórico](concorrencia.md), [entidades](entidades/README.md) | Retenção e purge não aprovados |
| `DB-07`, `DB-08`, `DB-10`, `DB-12` | [ENT-012 a ENT-016](entidades/README.md), [CONTR-011 — Administração UDS](../contratos/CONTR-011-api-administrativa-uds.md) | Modelo administrativo e auditoria ainda em refinamento |
| `DB-09` | [CONTR-012 — Assinaturas de eventos](../contratos/CONTR-012-assinaturas-eventos.md) | Não há schema, transporte e operações aprovados |

## Restrições de segurança

O grupo Linux `absabia` é uma **proposta de acesso ao socket da aplicação**, distinta de `abioma_db`, previsto para o UDS PostgreSQL. Pertencer a um grupo não concede automaticamente autorização SQL ou administrativa. A divergência histórica entre `abioma` e `absabia` ainda deve ser resolvida em [Segurança e autorização](../arquitetura/ARQ-006-seguranca-autorizacao.md).

## Condição de conclusão

Somente alterar o estado das pendências mediante decisão técnica rastreável e atualização das fontes canônicas correspondentes. Nenhuma migração SQL, teste de banco ou configuração de produção foi executada por esta revisão.

A [lista geral de bloqueios do MVP](../referencia/REF-006-gate-mvp.md) é complementar: ela documenta o gate de especificação, não substitui estas doze decisões específicas de persistência.

## Rastreabilidade

Fonte histórica integral preservada: [Pendências do modelo físico de persistência](pendencias.md), com os IDs `DB-01` a `DB-12` e as restrições do desenho anterior.
