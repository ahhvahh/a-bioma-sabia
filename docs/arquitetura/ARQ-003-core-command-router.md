<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# ARQ-003 — Core e Command Router

**ID:** `ARQ-003` · **Origem:** `MOD-0001` · **Estado histórico:** `refined`  
**Implementação:** prevista, sem evidência de código na branch

## Objetivo

Receber comandos internos já autorizados, localizar a operação cadastrada e delegar a execução sem depender do transporte Telegram.

## Responsabilidades

- resolver comando dentro do catálogo do cliente;
- receber `request_id`, `client_id`, `principal_id`, `command`, `arguments`, `reply_context` e `received_at` conforme CTR-0001;
- preservar `request_id`, `client_id` e `reply_context` até a produção do resultado;
- receber argumentos já tokenizados como `string[]`;
- delegar à operação correspondente a validação semântica dos argumentos;
- rejeitar comando desconhecido;
- delegar trabalho imediato ou criação de job;
- produzir resultado interno convertível pelo adaptador de transporte;
- não executar shell nem acessar diretamente Bot API.

## Entradas e saídas

**Entrada:** Comando interno validado pelo adaptador e pela camada de autorização.

**Saída:** Envelope conforme CTR-0001 com `type: message | job | file | error`, `request_id` e `payload` compatível com o tipo.

## Eventos

Pode solicitar criação de job ou execução de operação cadastrada.

## Persistência

Nenhuma persistência própria foi definida para o roteador.

## Restrições

- não depender de tipos específicos da Telegram Bot API;
- não conhecer caminhos de scripts informados pelo usuário;
- não conter lógica específica de FFmpeg ou ferramenta concreta.

## Critérios de aceite

- roteador pode ser testado com adaptadores mock;
- comando inexistente não chega a executor;
- o Router não interpreta texto bruto do transporte para obter argumentos;
- a operação correspondente valida quantidade, formato e domínio dos argumentos;
- `request_id` permanece estável durante roteamento e criação de job;
- o Router trata `reply_context.destination_id` como valor opaco;
- resultado imediato usa `message`, criação/referência assíncrona usa `job` e falhas controladas usam `error`;
- resultado `file` referencia `media_id` persistido e não carrega BLOB;
- novo executor pode ser integrado sem alterar o mecanismo de identificação de comando.

## Dependências e contratos

- [DSG-0002 — Componentes do Sabiá Core](ARQ-002-componentes-core.md)
- [ADR-0003 — Core independente do Telegram](../decisoes/DEC-003-core-independente-telegram.md)
- [CTR-0001 — Comando interno](../contratos/CONTR-001-comando-interno.md)
- [CTR-0002 — Execução de script](../contratos/CONTR-002-execucao-script.md)
- [CTR-0003 — Job](../contratos/CONTR-003-job.md)

## Implementação prevista

Prevista para o pacote interno responsável por comandos/roteamento. **Não verificada no código-fonte desta branch.**

## Pendências de integração

O roteador não executa shells nem comandos arbitrários: deve aplicar os contratos internos. A documentação descreve um módulo planejado; sua implementação não foi verificada.

## Rastreabilidade

[MOD-0001 — especificação original](../especificacao/modulos/core-command-router.md). O identificador original permanece válido para documentos históricos.
