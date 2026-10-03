# Registro do estado operacional

![FLW](https://img.shields.io/badge/FLW-FLW--0009-bf3989?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir o fluxo comum para converter uma informação de origem em estado PostgreSQL durável sem perder as garantias específicas de Telegram, jobs, mídia e alertas.

## Dependências

- [CTR-0010 — Origem e conversão](../contratos/origem-dados-persistencia.md)
- [PST-0100 — Entidades persistentes](../persistencia/entidades/README.md)
- [PST-0002 — Relacionamentos](../persistencia/relacionamentos-estado-operacional.md)

## Gatilhos

- update recebido do Telegram;
- criação/transição de job;
- resposta textual produzida;
- avaliação de alerta;
- upload integral por CTR-0008;
- abertura/chunk por CTR-0009;
- mudança de estado de transmissão.

## Fluxo principal

1. identificar a origem;
2. validar autorização da origem quando aplicável;
3. extrair somente campos definidos pelo contrato da origem;
4. normalizar por CTR-0010;
5. validar referências já persistidas necessárias;
6. calcular campos derivados;
7. iniciar transação curta;
8. gravar entidade principal e dependências que exigem atomicidade conjunta;
9. validar restrições/estado antes do commit;
10. commit;
11. somente depois emitir ACK, avançar cursor ou expor estado como durável.

## Regras específicas

### Telegram

`inbound_update` e avanço de `transport_cursor` pertencem à mesma unidade transacional. O offset remoto só pode avançar depois do commit.

### Requisição

A criação da `request` persiste o estado inicial `received`. Antes de entregar a requisição ao Core, o estado muda para `processing`.

Quando o Core produz seu resultado imediato, a request muda para `completed`. Resultado imediato inclui a criação persistida de um job assíncrono; a conclusão futura do job não altera `request.status`.

Erro controlado ou falha do processamento pelo Core muda a request para `failed`.

Estados de entrega de mensagem e mídia não alteram `request.status`.

### Job

Mudança de `job.status` e inserção em `job_state_history` pertencem à mesma transação.

### Resposta textual

A resposta é criada em `pending` antes do envio. O claim de PST-0003 persiste `pending → sending` antes da chamada externa. Confirmação remota e `delivered` são persistidas juntas; falha temporária retorna a mensagem para `pending`.

### Mídia integral

`media` e `media_transmission` são persistidas antes do ACK CTR-0008.

### Chunk

`media_chunk` e os contadores/estado da `media` são atualizados na mesma transação.

### Alertas

O novo estado deve ser durável para sobreviver a restart. A regra exata de atualização de `last_notified_at` permanece BLOCKED em PST-0107.

## Falhas

Falha antes do commit:

- não emitir sucesso;
- não avançar cursor;
- não avançar sequência;
- não marcar entrega como confirmada.

Falha depois do commit, antes do ACK:

- o estado persistido permanece fonte da verdade;
- regras de idempotência/retry do contrato específico são aplicadas.

## Resultado

Estado operacional durável e rastreável no PostgreSQL.

## BLOCKED

Este fluxo não pode atingir `refined` antes dos pontos BLOCKED de CTR-0010 e das entidades diretamente afetadas.
