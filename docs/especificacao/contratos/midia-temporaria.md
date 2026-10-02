# Mídia temporária por requisição

![CTR](https://img.shields.io/badge/CTR-CTR--0006-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir a representação de mídia de entrada e saída entre adaptadores, Core e processadores registrados sem transportar binários dentro do protocolo de controle.

## Dependências

- [ADR-0012 — Área temporária de mídia para processadores](../../adr/processamento/midia-temporaria-por-request.md)
- [CTR-0001 — Comando interno](comando-interno.md)
- [CTR-0005 — Protocolo de processador assíncrono](processador-assincrono.md)
- [MOD-0002 — Adaptador Telegram](../modulos/telegram.md)

## Tipo

`interface`

## Área temporária

A raiz de mídia do Sabiá é:

`/tmp/sabia/media`

Cada requisição possui:

`/tmp/sabia/media/<request_id>/`

Arquivos de entrada e de saída vinculados à requisição devem permanecer dentro desse diretório.

## Referência de mídia

A representação comum é:

- `name: string` — nome lógico/apresentável do arquivo;
- `path: string` — caminho do arquivo na área temporária da requisição.

`name` não é usado para resolver o caminho no filesystem.

`path` precisa apontar para arquivo regular existente e legível pelo Sabiá.

Antes de abrir o arquivo, o Sabiá deve resolver/normalizar o caminho e confirmar que ele permanece dentro de:

`/tmp/sabia/media/<request_id>/`

Caminhos relativos com escape, referências fora do diretório e links simbólicos que resolvam para fora da área da requisição devem ser rejeitados.

## Entrada de mídia

Quando o usuário envia imagem, vídeo ou outro arquivo suportado pelo adaptador:

1. o adaptador cria/usa o diretório da requisição;
2. baixa o conteúdo para a área temporária;
3. adiciona ao comando interno uma referência `attachment` com `name` e `path`;
4. o Core e o processador trabalham somente com essa referência local.

O contrato não transporta o binário da mídia como campo do comando interno.

## Saída de mídia

Processadores publicam arquivos por evento `content` de CTR-0005.

Cada evento referencia exatamente um arquivo:

- `request_id: string`;
- `status: content`;
- `content.name: string`;
- `content.path: string`.

Uma requisição pode publicar zero ou mais eventos `content`.

Cada evento pode ser encaminhado e entregue independentemente dos demais.

O Sabiá abre o arquivo validado e faz streaming para o adaptador Telegram sem exigir carregamento integral em memória.

## Lifecycle e limpeza

Para cada arquivo de saída anunciado por `content`:

1. validar a referência;
2. abrir o arquivo;
3. transmitir por streaming;
4. aguardar o retorno de sucesso do Telegram;
5. somente depois de o último byte ter sido transmitido e o Telegram confirmar o recebimento, remover imediatamente o arquivo.

Se o streaming ou a confirmação falhar, o arquivo permanece disponível para nova tentativa enquanto o processo atual continuar executando.

### Startup

Antes de aceitar novas requisições, o Sabiá remove todo o conteúdo existente em:

`/tmp/sabia/media`

A raiz pode ser recriada vazia em seguida.

Referências persistidas para arquivos temporários de uma execução anterior não são recuperáveis após essa limpeza.

### Shutdown

Antes de finalizar o processo, o Sabiá remove todo o conteúdo de:

`/tmp/sabia/media`

Entregas de mídia sem confirmação até esse ponto tornam-se não recuperáveis pela referência temporária e precisam permanecer registradas como falha de entrega, sem tentativa de reenvio no próximo startup.

Respostas textuais persistidas não são afetadas por essa política.

## Relação com finally

`content` não finaliza a requisição.

O processador pode enviar:

- zero ou mais `loading`;
- zero ou mais `content`;
- exatamente um `finally` válido para encerramento semântico normal.

`finally` contém a mensagem final e não transporta binário.

## Erros

- `media_path_outside_request` — caminho resolve fora do diretório da requisição;
- `media_not_found` — arquivo inexistente;
- `media_not_regular` — referência não aponta para arquivo regular;
- `media_unreadable` — Sabiá não consegue abrir o arquivo;
- `media_delivery_failed` — falha ao transmitir o arquivo ao transporte externo;
- `media_unavailable_after_cleanup` — entrega persistida referencia mídia que foi removida pela política de startup/shutdown.

## Regras e restrições

- conteúdo binário não entra no protocolo de controle;
- arquivos de requisições diferentes não compartilham diretório;
- o processador não pode solicitar envio de arquivo fora da área da própria requisição;
- um evento `content` referencia um único arquivo;
- múltiplos arquivos usam múltiplos eventos `content`;
- o streaming ao Telegram é responsabilidade do Sabiá;
- o arquivo não pode ser apagado antes da transmissão completa e confirmação de recebimento;
- confirmação bem-sucedida exige remoção imediata do arquivo;
- startup e shutdown limpam toda a raiz temporária;
- mídia temporária não é recuperável entre execuções do serviço;
- o binário não pode ser registrado em logs.

## BLOCKED

Ainda precisam ser definidos antes de `refined`:

- determinação normativa do tipo de mídia usado para escolher envio como imagem, vídeo ou documento;
- limites máximos de arquivo aceitos pelo Sabiá e pelos processadores.

## Compatibilidade

A referência `{name, path}` é independente do Telegram e pode ser usada por outros transportes locais que consigam acessar a mesma área temporária.

## Critérios de aceite

- mídia recebida pode ser disponibilizada ao processador como `{name, path}`;
- vários arquivos podem ser publicados pela mesma requisição;
- cada arquivo é transmitido por streaming;
- nenhum caminho fora do diretório da requisição é aceito;
- `content` não encerra o job;
- `finally` não carrega conteúdo binário;
- arquivo é removido somente após último byte transmitido e confirmação do Telegram;
- startup e shutdown limpam toda `/tmp/sabia/media`;
- mídia pendente removida pela limpeza não é reenviada após restart;
- tipo de mídia e limites ainda precisam ser refinados antes do gate.
