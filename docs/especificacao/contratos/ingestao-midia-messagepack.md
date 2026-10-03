# Ingestão de mídia por Unix socket e MessagePack

![CTR](https://img.shields.io/badge/CTR-CTR--0008-9a6700?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)

## Objetivo

Definir o contrato local usado por scripts, aplicações e serviços para enviar conteúdo binário ao Sabiá sem depender de Telegram, filesystem temporário ou objetos internos da aplicação.

## Dependências

- [ADR-0012 — Ingestão persistente de mídia por Unix socket](../../adr/processamento/ingestao-midia-socket-messagepack.md)
- [CTR-0006 — Mídia persistida](midia-persistida.md)
- [CTR-0007 — Transmissão persistente de mídia](transmissao-midia.md)
- [CTR-0009 — Ingestão fracionada de mídia por Unix socket](ingestao-midia-fracionada.md)
- [MOD-0006 — Segurança e autorização](../modulos/seguranca.md)

## Tipo

`mensagem`

## Transporte

O canal é um **Unix domain socket** local dedicado exclusivamente à ingestão de mídia.

Não existe listener TCP correspondente.

O caminho do socket é configuração obrigatória e deve ser absoluto.

O acesso ao socket é restringido pelas permissões do sistema operacional. A identidade Linux autorizada, ownership e modo concretos ainda precisam ser fechados antes de `refined`.

## Codificação

Cada conexão envia exatamente **um objeto MessagePack** de upload e recebe exatamente **um objeto MessagePack** de resposta.

Depois da resposta, a conexão pode ser encerrada.

Essa regra elimina necessidade de delimitador textual ou framing adicional para o MVP.

## Entrada

Objeto MessagePack:

- `version: integer` — versão do contrato; MVP usa `1`;
- `request_id: string` — UUID v4 gerado pelo Sabiá, representado como string canônica e conhecido pelo produtor;
- `name: string` — nome lógico do arquivo;
- `content_type: string | null` — tipo declarado pelo produtor, quando conhecido;
- `data: binary` — conteúdo binário integral.

Regras:

- antes de persistir, o Sabiá consulta o estado operacional e `request_id` deve corresponder a uma requisição conhecida;
- o Sabiá não tenta deduplicar uploads por nome, conteúdo ou `request_id`;
- `name` não representa caminho de filesystem;
- o produtor não fornece `client_id`, `transport` nem destino; esses dados são recuperados pela correlação do `request_id`;
- `data` nunca é escrito em logs;
- uma conexão envia um arquivo;
- vários arquivos para a mesma requisição usam várias conexões/uploads com o mesmo `request_id`;
- dois uploads com mesmo `request_id`, mesmo `name` e mesmo conteúdo são aceitos como duas mídias independentes e recebem `media_id` distintos.

## Limite de tamanho

O canal simples aceita no máximo **20 MB decimais**, equivalentes a `20000000` bytes, por upload.

O limite se aplica ao campo `data` de uma única mensagem MessagePack.

Conteúdo que exceda esse valor deve ser recusado com `media_too_large` antes de iniciar a persistência do BLOB.

Arquivos acima de 20 MB devem usar CTR-0009. Arquivos de até 20 MB também podem optar por CTR-0009 quando o produtor preferir ingestão fracionada.

## Persistência e atomicidade

Antes de confirmar o upload, o Sabiá deve:

1. validar a estrutura MessagePack;
2. validar a versão;
3. consultar o estado operacional e localizar a requisição por `request_id`;
4. rejeitar o upload com `media_too_large` se `data` exceder `20000000` bytes;
5. inserir metadados e BLOB da mídia no PostgreSQL;
6. criar a transmissão pendente correlacionada ao cliente/destino da requisição;
7. confirmar a transação.

O ACK de sucesso só pode ser enviado depois do commit da mídia e da transmissão.

Falha antes do commit não pode retornar sucesso.

Não existe chave de idempotência de upload no MVP. Se o produtor repetir um upload após perda do ACK, o novo recebimento é tratado como nova mídia.

## Saída de sucesso

Objeto MessagePack:

- `status: "accepted"`;
- `request_id: string`;
- `media_id: integer`.

`media_id` identifica o conteúdo persistido no Sabiá.

## Saída de erro

Objeto MessagePack:

- `status: "error"`;
- `code: string`;
- `message: string`.

Códigos mínimos:

- `unsupported_version`;
- `unknown_request`;
- `invalid_message`;
- `media_too_large`;
- `persistence_failed`;
- `not_authorized`.

## Segurança

- socket disponível apenas localmente;
- acesso depende de permissão do objeto Unix socket;
- credenciais e role PostgreSQL devem permanecer acessíveis somente à identidade operacional autorizada do Sabiá;
- conteúdo binário não é incluído em logs, auditoria ou mensagens de erro;
- nome e content type fornecidos pelo produtor não são tratados como dados confiáveis para decidir autorização;
- conhecer um `request_id` não substitui a autorização local para abrir o socket.

## Compatibilidade

Alteração incompatível do envelope exige nova versão.

Receptores devem rejeitar versões desconhecidas em vez de interpretar parcialmente o payload.

## BLOCKED

Antes de `refined` ainda precisam ser definidos:

- caminho normativo do socket simples;
- ownership, grupo e modo de acesso do socket simples.

## Critérios de aceite

- produtor local autorizado consegue enviar um BLOB MessagePack;
- upload com `request_id` desconhecido é rejeitado;
- sucesso só ocorre depois de persistência atômica;
- vários arquivos podem usar o mesmo `request_id`;
- uploads repetidos não são deduplicados e podem gerar `media_id` distintos;
- restart após ACK não perde o conteúdo persistido;
- nenhum caminho de filesystem é aceito no payload;
- binário não aparece em logs;
- `data` com mais de `20000000` bytes é recusado antes da persistência.
