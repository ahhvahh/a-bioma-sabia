<!-- A-BIOMA-DOC-OBSOLETO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Pendências do modelo físico de persistência

> **Conteúdo lido e migrado para:** [PERS-002 — Pendências do modelo físico](PERS-002-pendencias-modelo-fisico.md). As 12 decisões `DB-01` a `DB-12` permanecem abertas; este arquivo é conservado como origem histórica.

![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square) ![Versão](https://img.shields.io/badge/Vers%C3%A3o-proposta_v0.1-6e7781?style=flat-square)

O banco lógico `abioma_sabia`, schema `sabia` e nomes das roles são **propostas** desta revisão. A adoção operacional exige validação e autorização.

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

**Não confundir:** `absabia` dá acesso ao socket do Sabiá; `abioma_db` restringe o socket SQL. A senha/role SQL ou UDS não devem ser expostos aos operadores.

Outros bloqueios: [pendências gerais](../pendencias.md) e [regras consolidadas](../regras-consolidadas-sabia.md).
