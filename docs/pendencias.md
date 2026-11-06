<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Pendências documentais

> **Conteúdo lido e migrado para:** [REF-006 — Gate de especificação](referencia/REF-006-gate-mvp.md). O original permanece para rastreabilidade histórica.

Este documento consolida somente as lacunas que ainda impedem considerar o gate completo de especificação do MVP satisfeito.

Decisões já fechadas permanecem nos ADRs, contratos, fluxos e especificações correspondentes e não são repetidas aqui.

## Estado do gate

**BLOCKED**

ADRs necessários ao MVP estão `refined` e os desenhos estão `finalized`. O bloqueio atual está na etapa de especificação: ainda existem contratos, fluxos, configuração e partes do modelo persistente em `refinement`.

## BLOCKED

### Configuração operacional — CFG-0001

Documentos:

- [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md)
- [ADR-0009 — Persistência do estado operacional](adr/persistencia/estado-operacional.md)
- [CTR-0008 — Ingestão de mídia](especificacao/contratos/ingestao-midia-messagepack.md)
- [CTR-0009 — Ingestão fracionada](especificacao/contratos/ingestao-midia-fracionada.md)

Falta definir:

- schema concreto da conexão PostgreSQL, incluindo endpoint, TLS e referência de credencial;
- caminhos normativos dos dois Unix sockets de mídia;
- schema final de `schedules`, especialmente política de relatório e semântica de primeiro disparo/cadência;
- schema final de `logging`, incluindo valores aceitos e defaults necessários;
- estrutura normativa de comandos/operações habilitados por cliente, dependente dos contratos dos comandos do MVP.

### Telegram — normalização de mídia recebida

Documentos:

- [CTR-0010 — Origem e conversão dos dados persistentes](especificacao/contratos/origem-dados-persistencia.md)
- [PST-0108 — media](persistencia/entidades/media.md)
- [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)
- [FLW-0001 — Comando Telegram](especificacao/fluxos/comando-telegram.md)

Já estão definidos os caminhos brutos de `update_id`, usuário, chat, mensagem, texto e os `file_id` de foto/vídeo/documento.

Falta definir:

- qual item de `Update.message.photo[]` é escolhido quando houver múltiplos tamanhos;
- de-para final de `name` para mídia Telegram;
- de-para final de `content_type` quando o objeto recebido não fornecer MIME explícito.

### Persistência — relacionamento `inbound_update/request`

Documentos:

- [PST-0002 — Relacionamentos](persistencia/relacionamentos.md)
- [PST-0102 — inbound_update](persistencia/entidades/inbound-update.md)
- [PST-0103 — request](persistencia/entidades/request.md)

Falta decidir a direção física final da relação. O modelo atual contém a proposta de manter somente `request.inbound_update_id`, mas isso ainda não é decisão normativa.

### Persistência — consistência de `media_transmission`

Documentos:

- [PST-0002 — Relacionamentos](persistencia/relacionamentos.md)
- [PST-0110 — media_transmission](persistencia/entidades/media-transmission.md)

É obrigatório impedir que uma transmissão possua `request_id` diferente da mídia referenciada. Falta decidir se a garantia será implementada por restrição no PostgreSQL ou somente pela aplicação.

### Scheduler — correlação das mensagens de alerta

Documentos:

- [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md)
- [PST-0106 — outbound_message](persistencia/entidades/outbound-message.md)
- [PST-0001 — Tabelas do estado operacional](persistencia/readme.md)

O Scheduler produz notificações sem uma `request` de origem documentada, enquanto `outbound_message.request_id` é obrigatório.

Falta decidir o modelo de correlação. A documentação não deve criar request sintética, tornar `request_id` opcional ou adicionar outra chave sem decisão explícita.

### Scheduler — estado de alertas

Documentos:

- [PST-0107 — alert_state](persistencia/entidades/alert-state.md)
- [FLW-0003 — Monitoramento agendado](especificacao/fluxos/monitoramento-agendado.md)

Falta decidir:

- se `last_notified_at` é atualizado quando a entrega é criada ou somente após confirmação remota;
- o que fazer com `alert_state` quando o `schedule_id` deixa de existir no YAML.

### Persistência — migração e versionamento

Documentos:

- [PST-0111 — schema_version](persistencia/entidades/schema-version.md)
- [PST-0001 — Tabelas do estado operacional](persistencia/readme.md)
- [CFG-0001 — Modelo de configuração](especificacao/configuracao/modelo-configuracao.md)

PostgreSQL e migração no startup já estão definidos conceitualmente. Falta escolher o mecanismo concreto de migração/versionamento e, consequentemente, decidir se `schema_version` é tabela própria ou responsabilidade da ferramenta adotada.

### Persistência — retenção e purge

Documentos:

- [PST-0002 — Relacionamentos](persistencia/relacionamentos.md)
- [FLW-0009 — Registro do estado operacional](especificacao/fluxos/registro-estado-operacional.md)
- [PST-0103 — request](persistencia/entidades/request.md)
- [PST-0105 — job_state_history](persistencia/entidades/job-state-history.md)

A forma de exclusão já está definida: FKs operacionais usam `ON DELETE RESTRICT` e o purge remove dependências explicitamente das folhas para a raiz.

Falta definir quando registros concluídos podem ser removidos e quais históricos/metadados devem ser preservados.

### Persistência — dado incompatível no consumo

Documento:

- [FLW-0010 — Consumo do estado operacional](especificacao/fluxos/consumo-estado-operacional.md)

Falta definir o comportamento quando um valor persistido não pertence mais ao domínio esperado após migração ou inconsistência. O consumidor não pode decidir silenciosamente entre ignorar, corrigir, falhar ou colocar o item em estado de erro.

### Comandos do MVP — contratos operacionais

Documentos:

- [REQ-0001 — Escopo do primeiro MVP](especificacao/requisitos/mvp.md)
- [MOD-0001 — Core e Command Router](especificacao/modulos/core-command-router.md)
- [MOD-0002 — Adaptador Telegram](especificacao/modulos/telegram.md)

O Router genérico está `refined`, mas os comandos do MVP ainda não possuem contratos operacionais suficientes.

Para os comandos aplicáveis, falta definir:

- sintaxe e argumentos;
- vínculo com operação/script cadastrado;
- validações;
- resposta de sucesso;
- erros controlados;
- comportamento específico de `/run`, `/jobs` e `/job <id>`.

### CTR-0004 — Auditoria e logs

Documento:

- [CTR-0004 — Auditoria e logs](especificacao/contratos/auditoria-logs.md) — `refinement`

Falta definir:

- tipos dos campos;
- campos obrigatórios por família de evento;
- nomes/identificadores estáveis dos eventos auditáveis;
- resultados/estados permitidos;
- tratamento normativo de identificadores potencialmente sensíveis além da proibição já existente de segredos e BLOBs.

## Pendência condicional

### CTR-0003 — retry automático

Não bloqueia o MVP enquanto `max_retries = 0`.

Antes de habilitar retry automático ainda será necessário especificar `job_attempt`, incluindo identidade da tentativa, ordem, estado, falha e consumo de `max_retries`.

## Correções autônomas aplicadas nesta revisão

Sem criar decisões novas, foram reconciliados:

- ADR-0007 com ADR-0009: persistência dos alertas entre reinícios é PostgreSQL;
- FLW-0004 promovido a `refined`, pois o fluxo de shutdown já estava completo;
- MOD-0001 promovido a `refined`, removendo dependência obsoleta de lacunas já fechadas em CTR-0006;
- caminhos brutos básicos do Telegram consolidados em CTR-0010/PST-0103;
- `outbound_message.content` definido como texto já materializado pelo adaptador; arquivos permanecem em `media_transmission`;
- CTR-0004 corrigido de `refined` para `refinement`;
- motivos de falha de FLW-0005 alinhados ao código `transport_failure` de CTR-0005;
- dependência duplicada removida de MOD-0007;
- `transport_cursor`, `job` e `media_chunk` promovidos a `refined` por não possuírem lacuna própria;
- referências obsoletas no desenho DSG-0002 reconciliadas;
- detalhes físicos já definidos de IDs/tabelas e a genericidade de `transport_cursor` deixaram de ser tratados como blockers.

## Condição para liberar o MVP

O gate Especificação → Desenvolvimento permanece `BLOCKED` até que:

1. as decisões acima que alteram comportamento ou modelo sejam tomadas;
2. os documentos dependentes sejam atualizados para `refined`;
3. contratos e fluxos necessários ao escopo estejam `refined`;
4. não reste decisão arquitetural ou de negócio que precise ser tomada durante a implementação.


## Nova diretriz de acesso local e `sabiactl` — 03/11/2026

**Requisito novo, ainda não implementado:** serviço `sabia` isolado; APIs locais UDS acessíveis ao grupo `absabia`; conexão do serviço ao PostgreSQL exclusivamente por UDS protegido pelo grupo `abioma_db`; cliente de terminal `sabiactl` com menus, status, contagens, testes, configurações e upload/validação/ativação de scripts.

**Refinamentos pendentes:** harmonizar grupo legado `abioma` em CFG-0001/CTR-0008/CTR-0009/MOD-0006; definir caminhos e formatos dos UDS, autorização por operação, contratos das métricas e menus, protocolo para upload/validação/ativação de scripts e modelo persistente de catálogo; definir autenticação SQL (`pg_hba.conf`), diretório/socket PostgreSQL, role e bloqueio de conexões TCP indevidas. A pertença a `absabia` deve permitir conectar, mas mudanças sensíveis e ativação de scripts requerem autorização própria.

Ver [README](../README.md), [arquitetura](arquitetura.md), [segurança](seguranca-acessos.md) e [interfaces](interfaces-servicos.md).


## Fila de solicitações de acesso por cliente — 03/11/2026

**Nova experiência desejada para o `sabiactl`, ainda não implementada:** selecionar bot, listar múltiplas solicitações criadas pelo fluxo `/start`, navegar por `↑/↓`, rejeitar por `←`, aprovar por `→`, exigir `Y` para confirmar **qualquer** decisão e aceitar `N/Esc` para cancelar. A rejeição sai da fila mantendo auditoria. Não se trata de solicitação nativa `chat_join_request` do Telegram nem de assinatura paga.

**Bloqueios a refinar:** definir esquema de pedidos e autorizações por cliente no PostgreSQL, migração e política de rejeição/reenvio; compatibilidade com `allowed_users` no CFG-0001 (hoje no YAML com restart); operações e respostas UDS administrativas de listagem e decisão, paginação, autenticação do operador por bot, concorrência entre operadores, auditoria e notificações de decisão. Conferir [desenho da experiência](https://github.com/ahhvahh/a-bioma-sabia/blob/develop/docs/solicitacoes-acesso.md), [interfaces](interfaces-servicos.md) e [persistência](persistencia/readme.md).
