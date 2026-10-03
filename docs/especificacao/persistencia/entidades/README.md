# Entidades persistentes

**ID:** PST-0100  
**Status:** refinement

## Objetivo

Indexar as entidades propostas para o estado operacional PostgreSQL do Sabiá.

Cada documento de entidade detalha:

- finalidade da tabela;
- campos e tipos PostgreSQL;
- nulabilidade;
- origem de cada informação;
- campo de origem quando já documentado;
- conversão origem → persistência;
- exemplo ilustrativo de registro;
- regras de gravação e consumo;
- lacunas ainda `BLOCKED`.

## Dependências

- [PST-0001 — Proposta de tabelas](../tabelas-estado-operacional.md)
- [PST-0002 — Relacionamentos](../relacionamentos-estado-operacional.md)
- [CTR-0010 — Origem e conversão dos dados persistentes](../../contratos/origem-dados-persistencia.md)
- [FLW-0009 — Registro do estado operacional](../../fluxos/registro-estado-operacional.md)
- [FLW-0010 — Consumo do estado operacional](../../fluxos/consumo-estado-operacional.md)

## Entidades

- [PST-0101 — transport_cursor](transport-cursor.md)
- [PST-0102 — inbound_update](inbound-update.md)
- [PST-0103 — request](request.md)
- [PST-0104 — job](job.md)
- [PST-0105 — job_state_history](job-state-history.md)
- [PST-0106 — outbound_message](outbound-message.md)
- [PST-0107 — alert_state](alert-state.md)
- [PST-0108 — media](media.md)
- [PST-0109 — media_chunk](media-chunk.md)
- [PST-0110 — media_transmission](media-transmission.md)
- [PST-0111 — schema_version](schema-version.md)

Os valores usados nos exemplos são apenas ilustrativos e não definem formato normativo para IDs ou conteúdo.
