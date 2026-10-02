# Ingestão persistente de mídia por Unix socket

![ADR](https://img.shields.io/badge/ADR-ADR--0012-7a3e9d?style=flat-square)
![Status](https://img.shields.io/badge/Status-refined-0969da?style=flat-square)
![Version](https://img.shields.io/badge/Version-5-6e7781?style=flat-square)

## Contexto

O Sabiá precisa receber imagens, vídeos e outros arquivos produzidos por scripts, aplicações e serviços assíncronos e entregá-los posteriormente ao cliente correto.

A estratégia anterior baseada em arquivos temporários no filesystem não oferece durabilidade suficiente para recovery e exige coordenação de lifecycle entre banco e diretório temporário.

## Problema

É necessário receber conteúdo binário de produtores locais de forma controlada, correlacioná-lo a uma requisição existente, persistir o material antes de confirmar o recebimento e permitir que transmissões ao Telegram sejam retomadas após restart sem depender de arquivos temporários.

## Restrições

- somente processos locais autorizados devem alcançar o canal;
- o produtor precisa conhecer o `request_id` fornecido pelo Sabiá;
- o Core não deve receber objetos específicos da Telegram Bot API;
- o conteúdo só pode ser confirmado ao produtor depois de persistido;
- o binário não pode aparecer em logs;
- o protocolo precisa ser simples de implementar e testar;
- imagens e vídeos precisam ser suportados.

## Opções consideradas

### Área temporária no filesystem

Permite streaming simples, mas cria perda de durabilidade se o arquivo desaparecer e exige reconciliação entre filesystem e estado persistente.

### Unix socket local + MessagePack + BLOB persistido

O produtor envia metadados e binário por um socket local. O Sabiá valida a correlação e persiste o conteúdo no SQLite antes de responder com sucesso.

## Decisão

O Sabiá disponibilizará um **Unix domain socket para ingestão simples de mídia**. ADR-0013 complementa esta decisão com um segundo socket dedicado à ingestão fracionada.

O protocolo de aplicação desse socket será **MessagePack**.

No canal simples, cada envio representa um único item de mídia integral e contém, no mínimo:

- versão do contrato;
- `request_id`;
- nome lógico do arquivo;
- metadados de tipo quando informados;
- conteúdo binário.

O canal simples aceita até `20000000` bytes por arquivo. Conteúdo maior usa o canal fracionado de ADR-0013.

O Sabiá valida o `request_id`, persiste os metadados e o conteúdo binário no SQLite e somente depois retorna confirmação contendo o `media_id` persistente.

Uma mesma requisição pode receber vários arquivos por várias mensagens independentes no socket.

O banco de dados é a fonte oficial do conteúdo recebido. O envio ao Telegram lê o binário persistido e não depende de arquivo temporário.

Nenhuma área temporária de filesystem faz parte do contrato de transporte de mídia.

## Justificativa

A solução concentra durabilidade e correlação no mesmo armazenamento usado pelo estado operacional, elimina dependência de arquivos temporários e torna o protocolo de produtores locais independente do Telegram.

MessagePack oferece representação binária nativa e estrutura de mensagem compacta sem exigir serialização textual do conteúdo.

## Consequências

- o SQLite passa a armazenar BLOBs de mídia;
- será necessário um contrato de ingestão MessagePack;
- o socket precisa de política explícita de caminho, ownership e permissões locais;
- transmissões pendentes passam a referenciar `media_id`, não caminho de filesystem;
- mídia recebida do Telegram também pode usar o mesmo modelo persistente;
- o canal simples é limitado a 20 MB;
- conteúdos maiores usam o segundo socket e chunks de até 5 MB conforme ADR-0013;
- a retenção do BLOB após entrega precisa ser definida.

## Dependências

- [ADR-0009 — Persistência do estado operacional](../persistencia/estado-operacional.md)
- [ADR-0011 — Processadores assíncronos registrados e transporte de progresso](processadores-assincronos-registrados.md)
- [DSG-0002 — Componentes do Sabiá Core](../../desenho/componentes-core.md)
- [ADR-0013 — Dois canais de ingestão de mídia e upload fracionado](ingestao-midia-fracionada.md)

## Critérios de validação

- produtor local consegue enviar mídia usando MessagePack;
- envio só recebe ACK de sucesso depois do commit persistente;
- vários arquivos podem ser associados ao mesmo `request_id`;
- restart do serviço não perde mídia já confirmada ao produtor;
- transmissão ao Telegram usa conteúdo persistido por `media_id`;
- binário não depende de `/tmp`;
- acesso ao socket é limitado por controles locais do sistema operacional.
