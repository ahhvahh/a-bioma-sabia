<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# Guia do usuário — `sabiactl` e Telegram

**Situação desta revisão:** guia de interação **proposta**; comandos, telas administrativas e autorizações ainda dependem de implementação. Links para três referências históricas ausentes foram explicitados, sem criar conteúdo presumido.

> **As telas a seguir são propostas, não capturas de software executável.** Não há `sabiactl` na branch `develop`. O programa de terminal será destinado aos usuários Linux do grupo `absabia` e fará todas as solicitações pelo socket UDS do serviço, nunca pelo PostgreSQL diretamente.

## Terminal administrativo `sabiactl` — navegação planejada

### Tela inicial

```text
SABIÁ | Administração local       [EXEMPLO PLANEJADO]

Serviço: —       Conexões UDS: —
Sessões locais: —     Contatos Telegram: —
Clientes configurados: —  Processadores: —

1. Estado e conexões
2. Usuários e clientes
3. Testar serviços e integrações
4. Menus e ações
5. Configuração
6. Scripts (enviar, validar, ativar)
0. Sair

Escolha: _
```

### Solicitações de acesso — fila por cliente

O administrador escolhe um bot e usa `↑`/`↓` para percorrer as solicitações pendentes. `→` prepara a aprovação; `←` prepara a rejeição/retirada da fila. **Toda decisão exige `Y` para confirmar**; `N` ou `Esc` cancela. A solicitação recusada sai da fila, mas permanece registrada para auditoria.

As telas administrativas de solicitação de acesso ainda não possuem documento recuperado nesta branch. O [modelo de solicitação de acesso](persistencia/entidades/telegram-access-request.md) descreve apenas a persistência proposta, não o comportamento da interface.

## Como cadastrar um script e disponibilizar uma ação no bot

> **Fluxo planejado para o `sabiactl`: as telas, atalhos e confirmações abaixo ainda não são executáveis.** A referência histórica `FLW-007` não está disponível nesta branch; a [entidade proposta de catálogo de ações](persistencia/entidades/action-catalog.md) descreve apenas os vínculos de persistência, não o fluxo completo. A especificação prevê scripts cadastrados e ações persistidas; o **upload com validação prévia e a gestão pelo `sabiactl`** são requisitos novos, cujo contrato administrativo ainda precisa ser implementado.

A sequência para o operador será: **selecionar arquivo → verificar → enviar para área de validação → aprovar cadastro do script → escolher bot → cadastrar ação → publicar**. Se o script já tiver sido validado e cadastrado, é possível **pular o envio** e simplesmente selecioná-lo ao criar uma nova ação.

### 1. Preparar e selecionar o arquivo

Na tela inicial do `sabiactl`, escolha **6. Scripts (enviar, validar, ativar)** e depois **Enviar novo script**. O arquivo será escolhido **no computador onde o operador está executando o terminal**; o Sabiá não aceitará um caminho de escrita escolhido pelo usuário para seus diretórios internos.

```text
SABIÁ > Scripts > Cadastrar arquivo              [TELA PROPOSTA]

Arquivo local:       /home/operador/scripts/cpu.sh
Identificador:       consultar-cpu
Descrição:           Consulta de uso de CPU
Tipo:                Bash
Execução:            [ ] Convencional  [x] Assíncrona
Interpretador:       /usr/bin/bash
Timeout:             2h (processador assíncrono)
Ambiente permitido:  nenhum

[ V ] Verificar arquivo   [ Enter ] Continuar   [ Esc ] Voltar
```

O nome, o arquivo e o identificador são **exemplos**, não caminhos obrigatórios. O modo assíncrono do MVP aceita scripts Bash. Scripts usados como processadores assíncronos precisam seguir o protocolo de progresso descrito adiante.

### 2. Validar antes de aceitar o script

A verificação será dividida em duas partes. **O arquivo precisa ser transmitido ao serviço para que a validação do servidor seja feita; isso não significa que já foi aceito ou instalado.**

| Fase | Verificações previstas | Resultado |
|---|---|---|
| Antes do envio, no `sabiactl` | Arquivo existente, legível, regular, não vazio; tamanho dentro do limite; identificação e extensão/tipo compatíveis | Se falhar, **não enviar** |
| Após o envio, no Sabiá | Integridade do conteúdo, tipo permitido, formato, sintaxe Bash (por exemplo, `bash -n`), definição e existência de interpretador permitido, regras de autorização, consistência dos metadados e conformidade com o contrato do processador | Se falhar, **rejeitar sem incorporar** |
| Teste controlado, quando exigido | Testar o comportamento em ambiente isolado e sem segredos/acessos privilegiados, verificando eventos `loading` e `finally`, timeout e erros de saída | Somente liberar se os testes obrigatórios passarem |

**A verificação de sintaxe não garante segurança.** O serviço não poderá executar automaticamente um script recém-enviado com os privilégios do daemon para tentar validá-lo. Testes que executem código exigem isolamento específico e autorização; sem esse ambiente, o arquivo permanece **aguardando revisão**, não pode ser ativado.

Durante a validação, o Sabiá manterá o conteúdo em **área temporária não executável e isolada do catálogo ativo**. Somente o serviço poderá promover arquivos validados para o armazenamento interno definitivo. A especificação ainda precisa estabelecer limites exatos de tamanho, diretórios, testes obrigatórios e política de retenção de arquivos rejeitados.

```text
SABIÁ > Scripts > Resultado da validação         [TELA PROPOSTA]

Arquivo:      cpu.sh
Identificador: consultar-cpu
Integridade:  OK
Sintaxe Bash: OK
Contrato:     OK
Teste isolado: OK (quando habilitado e exigido)

Resultado:    VALIDADO — disponível para aprovação

[ Y ] Aceitar e cadastrar  [ N ] Cancelar  [ Esc ] Voltar
```

**Somente `Y` confirma o cadastro**, depois da validação completa. `N` ou `Esc` não incorporam o arquivo. Em caso de rejeição, a tela deve mostrar os erros específicos, permitir corrigir e reenviar e preservar os dados necessários para auditoria; nenhum arquivo rejeitado poderá ser selecionado por uma ação.

### 3. Conferir o script cadastrado

Após a aceitação, o Sabiá armazena o arquivo em uma pasta interna controlada e registra sua **identidade lógica e estado**. O operador poderá voltar à lista de scripts e verificar:

```text
SABIÁ > Scripts                                  [TELA PROPOSTA]

  ID                 Estado       Tipo        Ações
> consultar-cpu      Cadastrado   Bash/async  0
  gerar-relatorio    Cadastrado   Bash        2

[ Enter ] Detalhes   [ Esc ] Voltar
```

Um script **cadastrado** ainda não precisa aparecer no Telegram: ele só se tornará disponível por meio de uma **ação publicada e autorizada**. O cadastro também não concede autorização irrestrita para executá-lo.

### 4. Cadastrar uma ação e vinculá-la a um bot

No menu principal, escolha **4. Menus e ações → Selecionar bot → Nova ação**. Selecione um script já validado e cadastrado no serviço.

```text
SABIÁ > Menus e ações > Nova ação                 [TELA PROPOSTA]

Cliente Telegram:     bioma
Categoria do menu:    Sistema
Nome exibido:         Consultar CPU
Identificador da ação: cpu-status
Script vinculado:     consultar-cpu   [VALIDADO]
Modo:                 Assíncrono
Parâmetros aceitos:   Nenhum
Usuários autorizados: Conforme permissões do bot
Visibilidade:         Desabilitada (rascunho)

[ Enter ] Validar ação   [ Esc ] Voltar
```

O operador deverá informar **qual bot**, **qual menu/categoria**, **qual script cadastrado**, **quais argumentos o usuário poderá enviar** e **quem estará autorizado**. Para ações que recebem argumentos (por exemplo, uma URL), o serviço precisa validá-los contra regras explícitas; o Telegram nunca poderá fornecer o caminho do script, o interpretador ou o diretório de trabalho.

O Sabiá verifica se o bot existe e está habilitado, o script foi validado, o identificador da ação é único no escopo apropriado, o menu existe e os parâmetros e permissões são válidos. **Uma ação não pode ser publicada apontando para script rejeitado, inexistente ou não aprovado.**

### 5. Revisar e publicar a ação

```text
SABIÁ > Menus e ações > Confirmar publicação      [TELA PROPOSTA]

Bot:        bioma
Menu:       Sistema > Consultar CPU
Script:     consultar-cpu (validado)
Argumentos: nenhum
Acesso:     usuários autorizados do bot

Publicar esta ação?    Y = Sim   N / Esc = Cancelar
```

Apenas depois de pressionar **`Y`** o `sabiactl` solicita ao serviço a publicação. O Sabiá revalida o estado e a autorização, persiste o vínculo no PostgreSQL e passa a incluir a ação no catálogo do bot para os usuários autorizados. O `sabiactl` **não grava diretamente no banco** nem instala scripts por conta própria.

O mesmo script poderá atender a outras ações e bots, **desde que cada vínculo seja cadastrado explicitamente** e sua autorização seja avaliada no contexto do respectivo cliente Telegram.

### 6. O que o usuário verá no Telegram

```text
Sabiá
[ Sistema ] [ Ferramentas ]

Sistema
[ Consultar CPU ]
[ Voltar ] [ Início ]
```

Ao escolher **Consultar CPU**, o Sabiá resolve a ação autorizada, inicia o script cadastrado e apresenta o resultado. O menu só mostrará ações permitidas para aquele usuário e chat.

### Entrada e resposta do script

Os contratos existentes distinguem dois modos:

| Modo | Entrada | Saída |
|---|---|---|
| Convencional — [CTR-0002](contratos/CONTR-002-execucao-script.md) | Argumentos validados em `argv` | Texto livre em `stdout`, diagnóstico em `stderr` e código de saída |
| Processador assíncrono — [CTR-0005](contratos/CONTR-005-processador-assincrono.md) | `request_id` UUID v4 em uma linha de `stdin`; argumentos validados em `argv` | **Uma mensagem JSON Lines por linha em `stdout`**, com `request_id`, `status` (`loading` ou `finally`) e `message`; diagnóstico em `stderr` |

Exemplo de saída válida de um processador assíncrono (o `request_id` deve ser o mesmo que o Sabiá forneceu):

```json
{"request_id":"550e8400-e29b-41d4-a716-446655440000","status":"loading","message":"Consultando CPU"}
{"request_id":"550e8400-e29b-41d4-a716-446655440000","status":"finally","message":"Consulta concluída"}
```

No MVP, o timeout de processador assíncrono é de **2 horas**. Mídia binária não deve sair em `stdout`: usa os contratos próprios de ingestão UDS. A saída JSON Lines precisa estar correta; texto adicional em `stdout` viola o protocolo.

### Falhas que o operador poderá encontrar

| Situação | Comportamento esperado |
|---|---|
| Arquivo não encontrado, vazio ou ilegível | Bloquear o envio e informar o motivo |
| Sintaxe ou contrato inválido | Rejeitar o script, mostrar o erro e não cadastrá-lo |
| Teste inseguro ou isolamento indisponível | Manter aguardando revisão; não ativar |
| Usuário sem autorização administrativa | Negar cadastro/publicação mesmo que tenha acesso ao UDS |
| Script não cadastrado ou rejeitado | Impedir associação ou publicação de ação |
| Bot desabilitado ou menu inválido | Recusar publicação e permitir correção |
| Serviço ou PostgreSQL indisponível | Exibir erro sem presumir sucesso; nunca acessar o banco diretamente |
| Mudança concorrente por outro operador | Informar conflito e exigir nova revisão |
| Cancelamento com `N` ou `Esc` | Não aceitar script nem publicar ação |

**Falta implementar:** contratos de upload e validação administrativa, critérios seguros de teste, armazenamento e versionamento de scripts, catálogo persistente de ações e publicação dinâmica. A [configuração atual](referencia/REF-001-modelo-configuracao.md) ainda descreve registro de scripts no YAML carregado na inicialização; essa regra precisará ser conciliada com a administração pelo `sabiactl` antes de implementar o fluxo acima.


### Inspeção de conexões

```text
SABIÁ > Estado e conexões

Conexões UDS ativas: —
Sessões administrativas locais: —
Contatos Telegram cadastrados: —
Clientes Telegram configurados: —

1. Atualizar
2. Detalhar
0. Voltar
```

As contagens são distintas e não devem confundir contatos cadastrados com usuários online no Telegram. Os traços indicam valores não disponíveis no protótipo.

**Erros previstos:** sem grupo `absabia`, mostrar acesso negado; serviço indisponível, mostrar falha de conexão sem buscar o banco diretamente; lista vazia, mostrar ausência de registros; opção inválida, manter o usuário no menu. Os textos finais, teclas de navegação e regras por operação estão pendentes.

---

## Telegram — interação prevista

### Fluxos Telegram já especificados

> **Ainda não é um manual de aplicação disponível.** O repositório contém fluxos e exemplos de menu, mas não há adaptador Telegram implementado em `/src`. As telas abaixo são **referências da especificação**, não capturas reais ou comprovação de funcionamento.

## Entrada e navegação documentadas

No canal remoto, a interação ocorre no **Telegram**; no terminal local, o cliente será o `sabiactl` descrito acima. O fluxo de referência prevê:

1. Abrir a conversa com um cliente Telegram autorizado e enviar `/start` (ou `/menu`).
2. Receber o menu das ações permitidas ao usuário e ao chat.
3. Escolher uma categoria, navegar às ações e selecionar a desejada.
4. Informar apenas os parâmetros permitidos, se a ação exigir, e consultar progresso/resultado.
5. Escolher **Voltar** para a categoria anterior ou **Início** para a raiz.

### Tela de referência: menu inicial

```text
Sabiá

[ Automações ] [ Ferramentas ]
[ Status ]      [ Ajuda ]
```

Ao selecionar **Ferramentas**, o desenho de interação apresentado na documentação original usa o seguinte exemplo:

```text
Ferramentas

[ Converter vídeo ]
[ Voltar ] [ Início ]
```

`Converter vídeo` é **exemplo de catálogo**, não confirmação de que a função integra o primeiro MVP. As ações exibidas deverão depender do cadastro e da autorização. Os cliques serão tratados como `callback_query` e a seleção deverá ser revalidada pelo serviço.

## Comandos descritos para o MVP

| Cliente lógico previsto | Comandos documentados |
|---|---|
| Bioma | `/start`, `/help`, `/status`, `/cpu`, `/memory`, `/disk`, `/uptime`, `/run` |
| Tools | `/help`, `/jobs`, `/job` |
| Alerts | `/status` |

Esses comandos constam do escopo, mas seus argumentos, mensagens de retorno e regras operacionais permanecem incompletos. **Não é possível apresentar uma interação funcional reproduzível.**

## Exceções e restrições

| Situação | Comportamento especificado ou lacuna |
|---|---|
| Usuário conhecido, sem autorização | Contato não recebe permissão automaticamente; operação restrita deve ser recusada |
| Menu sem ações disponíveis | Texto e apresentação ainda não definidos |
| Seleção inválida ou menu antigo | Ação/callback deve ser revalidada; mensagens finais pendentes |
| Serviço ou PostgreSQL indisponível | Há falhas previstas; texto exato ao usuário não definido |
| Anexo ou resultado | Entrega planejada pelo Telegram; limites de mídia e formatos dependem de contratos e API |
| Busca, paginação ou telas de terminal | Não há interface implementada ou comportamento confirmado |

## Fontes da experiência prevista

- [Contrato operacional Telegram e exemplos de botões](referencia/REF-004-operacao-telegram.md)
- [FLUX-001 — Comando Telegram](fluxos/FLUX-001-comando-telegram.md) — fluxo disponível; não substitui integralmente o documento histórico `FLW-006`, ausente nesta branch
- `FLW-007` — referência histórica ausente nesta branch; ver [modelo proposto de ações](persistencia/entidades/action-catalog.md), que não substitui o fluxo administrativo
- [Escopo do MVP](referencia/REF-002-escopo-primeiro-mvp.md)
- [Pendências dos contratos operacionais](referencia/REF-006-gate-mvp.md)
