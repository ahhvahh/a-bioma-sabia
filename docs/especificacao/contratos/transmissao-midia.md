# Transmissão persistente de mídia

![CTR](https://img.shields.io/badge/CTR-CTR--0007-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)

## Objetivo

Definir o registro persistente de arquivos que precisam ser transmitidos ao cliente, permitindo retomar transmissões após reinício do serviço e remover com segurança arquivos órfãos da área temporária.

## Dependências

- [ADR-0009 — Persistência do estado operacional](../../adr/persistencia/estado-operacional.md)
- [ADR-0012 — Área temporária de mídia para processadores](../../adr/processamento/midia-temporaria-por-request.md)
- [CTR-0006 — Mídia temporária por requisição](midia-temporaria.md)
- [MOD-0002 — Adaptador Telegram](../modulos/telegram.md)

## Tipo

`interface`

## Registro persistente

Cada arquivo destinado a transmissão deve possuir registro no SQLite antes da primeira tentativa de envio.

Campos mínimos:

- `transmission_id: integer` — identificador persistente da transmissão;
- `request_id: string` — correlação com a requisição;
- `client_id: string` — cliente lógico responsável pela entrega;
- `transport: string` — transporte de destino;
- `destination_id: string` — destino opaco no transporte;
- `name: string` — nome apresentado ao cliente;
- `path: string` — caminho validado dentro de `/tmp/sabia/media/<request_id>/`;
- `status: pending | transmitting | delivered | failed`;
- `created_at: timestamp`;
- `updated_at: timestamp`;
- `remote_message_id: string | null` — identificador remoto quando houver confirmação;
- `last_error: string | null` — erro controlado da tentativa mais recente.

O tipo de mídia pode ser acrescentado quando CTR-0006 fechar a regra normativa de classificação como imagem, vídeo ou documento.

## Estados

### pending

Arquivo registrado e aguardando tentativa de transmissão.

### transmitting

Existe tentativa de envio em andamento ou a execução anterior terminou sem registrar conclusão definitiva.

No startup, `transmitting` é tratado novamente como elegível para envio. Essa regra preserva semântica `at-least-once`: uma falha ambígua pode resultar em duplicidade, mas não em descarte silencioso.

### delivered

O Telegram confirmou o recebimento e a confirmação remota foi persistida.

Depois de persistir `delivered`, o arquivo temporário deve ser removido imediatamente.

### failed

A transmissão não pode continuar automaticamente, por exemplo quando o arquivo referenciado deixou de existir.

## Ordem de persistência e envio

Para um evento `content` válido:

1. validar `request_id`, `name` e `path`;
2. confirmar que o arquivo pertence ao diretório da requisição;
3. persistir a transmissão como `pending`;
4. somente depois iniciar o streaming;
5. marcar `transmitting` ao iniciar a tentativa;
6. transmitir o arquivo;
7. receber confirmação do transporte;
8. persistir `remote_message_id` e `delivered`;
9. remover imediatamente o arquivo.

O arquivo nunca pode ser removido apenas porque o último byte foi escrito no socket. A remoção exige confirmação remota persistida.

## Recovery no startup

Antes de aceitar novas transmissões:

1. consultar registros `pending` e `transmitting`;
2. para cada registro, verificar se o arquivo ainda existe e permanece dentro da área permitida;
3. se existir, torná-lo elegível para nova tentativa;
4. se estiver ausente, marcar `failed` com erro `media_missing`;
5. depois da reconciliação, executar a coleta de órfãos.

O startup não limpa integralmente `/tmp/sabia/media`.

## Coleta de arquivos órfãos

Um arquivo pode ser removido pelo coletor somente quando todas as condições forem verdadeiras:

- está dentro de `/tmp/sabia/media`;
- não existe registro `pending` ou `transmitting` que referencie seu caminho;
- o arquivo está sem modificação há mais de **1 minuto**.

Para o MVP, a idade operacional é calculada pelo tempo desde o `mtime` do arquivo. Isso evita remover um arquivo que ainda esteja sendo produzido por um processador.

Diretórios vazios podem ser removidos após a coleta dos arquivos.

## Shutdown

O shutdown não remove arquivos referenciados por transmissões `pending` ou `transmitting`.

Esses arquivos permanecem no filesystem para recovery no próximo startup do serviço.

Arquivos órfãos podem ser coletados pelas mesmas regras de idade e ausência de referência persistente.

## Erros

- `media_missing` — registro ativo referencia arquivo inexistente;
- `media_path_invalid` — caminho não pertence à área da requisição;
- `media_persistence_failed` — falha ao registrar transmissão antes do envio;
- `media_delivery_failed` — falha de transporte;
- `media_confirmation_persistence_failed` — transporte confirmou, mas a confirmação não foi persistida.

## Regras e restrições

- nenhuma transmissão começa antes de existir registro `pending`;
- `pending` e `transmitting` impedem coleta do arquivo correspondente;
- reinício do serviço retoma `pending` e `transmitting`;
- confirmação remota é persistida antes da remoção do arquivo;
- arquivo órfão só é coletado após mais de 1 minuto sem modificação;
- conteúdo binário não é persistido no SQLite.

## Compatibilidade

O registro usa `transport` e `destination_id` para não depender exclusivamente do Telegram.

## Critérios de aceite

- restart do serviço retoma transmissões pendentes;
- transmissão em estado ambíguo pode ser reenviada;
- arquivo ativo não é removido pelo coletor;
- arquivo órfão sem modificação há mais de 1 minuto é removido;
- confirmação remota precede remoção do arquivo;
- arquivo ausente no recovery produz falha explícita.
