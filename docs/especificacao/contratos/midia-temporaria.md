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
- [CTR-0007 — Transmissão persistente de mídia](transmissao-midia.md)

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

O lifecycle de arquivos de saída segue CTR-0007.

Antes da primeira tentativa de envio, o Sabiá persiste uma transmissão com correlação suficiente para reenviar o arquivo ao cliente correto.

Arquivos em transmissões `pending` ou `transmitting` não podem ser removidos.

Após transmissão completa, confirmação remota e persistência do estado `delivered`, o arquivo é removido imediatamente.

### Startup

O startup não limpa toda a área temporária.

O Sabiá reconcilia transmissões `pending` e `transmitting` com o filesystem e tenta novamente os arquivos existentes.

Arquivos sem registro ativo podem ser removidos somente quando estiverem sem modificação há mais de 1 minuto.

### Shutdown

O shutdown preserva arquivos associados a transmissões `pending` ou `transmitting` para permitir recovery no próximo startup do serviço.

Arquivos órfãos podem ser coletados pelas mesmas regras de CTR-0007.

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
- `media_unavailable_after_cleanup` — registro ativo referencia mídia que desapareceu do filesystem.

## Regras e restrições

- conteúdo binário não entra no protocolo de controle;
- arquivos de requisições diferentes não compartilham diretório;
- o processador não pode solicitar envio de arquivo fora da área da própria requisição;
- um evento `content` referencia um único arquivo;
- múltiplos arquivos usam múltiplos eventos `content`;
- o streaming ao Telegram é responsabilidade do Sabiá;
- o arquivo não pode ser apagado antes da transmissão completa e confirmação de recebimento;
- confirmação bem-sucedida exige remoção imediata do arquivo;
- startup e shutdown preservam mídia referenciada por transmissões ativas;
- transmissões ativas podem ser retomadas após restart do serviço;
- órfãos sem modificação há mais de 1 minuto podem ser coletados;
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
- startup reconcilia e retoma mídia pendente quando o arquivo existe;
- shutdown preserva arquivos de transmissões ativas;
- arquivo órfão sem modificação há mais de 1 minuto pode ser removido;
- tipo de mídia e limites ainda precisam ser refinados antes do gate.
