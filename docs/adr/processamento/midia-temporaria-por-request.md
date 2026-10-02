# Área temporária de mídia para processadores

![ADR](https://img.shields.io/badge/ADR-ADR--0012-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-2-6e7781?style=flat-square)

## Contexto

Processadores registrados podem produzir uma ou várias imagens, vídeos ou outros arquivos durante uma requisição assíncrona. Transportar o conteúdo binário dentro do protocolo de controle aumenta complexidade, memória utilizada e acoplamento entre processador e transporte Telegram.

## Problema

É necessário permitir que scripts, aplicações e serviços produzam múltiplos arquivos por requisição e que o Sabiá faça o envio desses arquivos ao cliente sem carregar o binário no protocolo de progresso/finalização.

## Restrições

- arquivos precisam permanecer acessíveis ao processo do Sabiá;
- requisições concorrentes não podem colidir entre si;
- caminhos informados pelo processador não podem permitir leitura arbitrária fora da área controlada;
- arquivos podem ser grandes, especialmente vídeos;
- o envio ao Telegram deve permitir streaming a partir do arquivo.

## Opções consideradas

### Binário dentro das mensagens do processador

Aumenta o tamanho do protocolo de controle e pode exigir carregamento integral em memória.

### Área temporária compartilhada com referência por caminho

O processador grava o arquivo e publica uma referência; o Sabiá valida e faz streaming do arquivo ao transporte externo.

## Decisão

A raiz temporária de mídia do Sabiá é:

`/tmp/sabia/media`

Cada requisição usa um diretório isolado:

`/tmp/sabia/media/<request_id>/`

Processadores gravam seus arquivos de saída dentro do diretório correspondente ao `request_id`.

O protocolo assíncrono passa a aceitar o status `content`. Cada evento `content` referencia um único arquivo e contém:

- `request_id`;
- `status: content`;
- `content.name`;
- `content.path`.

Uma requisição pode publicar zero ou mais eventos `content`. Portanto, múltiplos arquivos são enviados como múltiplos eventos correlacionados ao mesmo `request_id`.

O Sabiá é responsável por abrir o arquivo referenciado e fazer streaming ao adaptador Telegram. O binário deixa de fazer parte do protocolo de controle.

`finally` permanece como marcador de encerramento semântico da requisição e pode conter apenas a mensagem final.

### Lifecycle da área temporária

Um arquivo anunciado por `content` permanece disponível enquanto sua entrega estiver em andamento ou puder ser repetida no processo atual.

O arquivo só pode ser removido após as duas condições ocorrerem:

1. o streaming transmitiu o último byte do arquivo ao Telegram;
2. o Telegram confirmou com sucesso o recebimento da mensagem/arquivo.

Após a confirmação, o Sabiá remove imediatamente o arquivo correspondente.

Ao iniciar, o Sabiá limpa integralmente `/tmp/sabia/media` antes de aceitar novas requisições.

Ao encerrar, o Sabiá limpa integralmente `/tmp/sabia/media` antes de finalizar o processo.

Consequentemente, arquivos temporários não são mecanismo de recovery entre execuções do Sabiá. Uma entrega de mídia ainda não confirmada no momento de shutdown/restart deixa de ser recuperável a partir dessa referência temporária.

## Justificativa

A referência por caminho reduz a complexidade do protocolo, evita transportar vídeos/imagens como payload de controle e permite que o Sabiá faça streaming diretamente do filesystem.

O isolamento por `request_id` reduz colisões de nomes e facilita validação e limpeza posterior.

## Consequências

- processadores precisam conhecer o diretório temporário da própria requisição;
- o Sabiá precisa validar que `content.path` pertence à área da requisição;
- arquivos podem ser entregues individualmente antes do `finally`;
- arquivos são removidos imediatamente após transmissão completa e confirmação do Telegram;
- startup e shutdown limpam toda a raiz temporária;
- mídia temporária não sobrevive como mecanismo de recovery entre execuções;
- o tipo de mídia ainda precisa ser determinado de forma normativa antes do envio ao Telegram.

## Dependências

- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](processadores-assincronos-registrados.md)
- [CTR-0005 — Protocolo de processador assíncrono](../../especificacao/contratos/processador-assincrono.md)

## Critérios de validação

- dois jobs simultâneos usam diretórios distintos;
- um processador pode produzir vários arquivos para a mesma requisição;
- cada arquivo é anunciado por um evento `content`;
- o binário não é carregado no protocolo de controle;
- o Sabiá consegue transmitir o arquivo por streaming;
- caminhos fora do diretório da requisição não são aceitos;
- arquivo confirmado pelo Telegram é removido imediatamente;
- startup e shutdown deixam `/tmp/sabia/media` vazia.
