# T01 — Inbox durável

- T01-01: falha antes do INSERT → offset não avança; update volta na próxima leitura.
- T01-02: INSERT feito, commit abortado → não confirma; registro não fica parcialmente aceito.
- T01-03: commit atômico confirmado, processo cai antes de novo polling → raw_update completo permite recuperação.
- T01-04: mesmo update_id recebido duas vezes → UNIQUE e claim impedem duas execuções.
- T01-05: dois workers disputam o mesmo update → só um vence claim.
- T01-06: callback_query após reinício → reconstruir ação a partir do update salvo.
- T01-07: update com mídia e file_id → após restart recupera metadata e solicita novo getFile se URL expirar.
- T01-08: processamento concluído e queda antes do envio → saída permanece pendente e não repete script.
- T01-09: script com efeito colateral não idempotente → exigir política de reconciliação; não repetir cegamente.
