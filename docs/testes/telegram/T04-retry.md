# T04 — Retentativas e recovery

- T04-01: getUpdates 5xx → backoff 1,2,4,8,16,30 s.
- T04-02: 429 com retry_after → respeitar prazo mínimo.
- T04-03: resposta getUpdates vazia e bem-sucedida → backoff reinicia.
- T04-04: erro de credencial → suspender só cliente afetado.
- T04-05: sendMessage com timeout ambíguo → mantém tentativa recuperável; aceita duplicação eventual da resposta.
- T04-06: envio confirmado, falha em gravar message_id → reconciliar mantendo semântica at-least-once.
- T04-07: restart com outbound sending → voltar a pending sob claim seguro.
