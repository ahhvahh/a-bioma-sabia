# Solicitações de acesso — operação pelo `sabiactl`

> **Projeto de interface, não funcionalidade implementada.** Esta página registra a proposta de navegação definida em 03/11/2026. A branch `develop` não contém o binário `sabiactl`, a API administrativa ou as tabelas de aprovação. O esquema de configuração atual ainda usa `allowed_users` no YAML, com leitura apenas no startup; será necessário reconciliá-lo para aplicar aprovações dinamicamente.

## Objetivo

Permitir que o administrador de um bot analise **múltiplas solicitações de acesso**, escolhendo o cliente lógico (bot) antes de aprovar ou rejeitar cada pessoa. O cadastro de contato e a concessão de acesso são operações distintas: receber `/start` **não** significa estar autorizado.

**Terminologia:** “solicitação de assinatura” aqui significa **pedido interno de acesso ao bot Sabiá**, e não assinatura paga da plataforma Telegram ou solicitação nativa de ingresso em grupo/canal (`chat_join_request`). A fila é criada e gerida pelo próprio Sabiá com os dados recebidos dos updates.

## Passo a passo previsto

1. O usuário abre seu bot e envia `/start`. O Sabiá identifica `client_id` (bot), `telegram_user_id` e `chat_id`, registra/atualiza contato e solicita acesso caso ainda não autorizado, sem duplicar pedido pendente.
2. O operador local abre `sabiactl` e escolhe **Usuários e clientes → Solicitações de acesso**.
3. Seleciona o bot desejado e visualiza a respectiva fila de pedidos `pending`.
4. Usa `↑` e `↓` para escolher a pessoa. Usa `→` para **acolher/aprovar** ou `←` para **excluir da fila/rejeitar**.
5. A aplicação abre uma confirmação com nome, Telegram User ID, bot e consequência. **Somente `Y` executa a decisão**; `N` ou `Esc` cancelam sem alterações.
6. O `sabiactl` envia a operação ao serviço pelo socket UDS administrativo. O Sabiá revalida identidade, bot, estado do pedido e autorização administrativa antes de persistir no PostgreSQL.
7. Em caso de sucesso, a solicitação sai da lista de pendentes. A aprovação concede acesso apenas ao bot selecionado; a rejeição mantém histórico auditável. Eventual aviso ao usuário é enfileirado pelo serviço e não altera a transação de autorização caso a entrega ao Telegram falhe.

A operação de rejeitar **não é exclusão física** do contato ou histórico; remove apenas da fila de pendências. A política de nova solicitação após rejeição continua a ser definida.

## Tela 1 — selecionar bot

```text
SABIÁ > Usuários e clientes > Solicitações de acesso

Selecione o cliente Telegram                       [PLANEJADO]

> bioma       12 solicitações pendentes
  tools        3 solicitações pendentes
  alerts       0 solicitações pendentes

↑/↓ Selecionar   Enter Abrir   Esc Voltar
```

Os números são **dados fictícios de demonstração**, não contagens reais. O cliente `alerts` com lista vazia deve mostrar “Nenhuma solicitação pendente”, sem permitir operações sobre uma linha inexistente.

## Tela 2 — fila por bot

```text
SABIÁ > Solicitações de acesso > bioma            [PLANEJADO]
Pendentes: 3                          Página: 1/1

   Usuário              Telegram User ID   Solicitado
>  Maria Oliveira       123456789          14:32
   João Pereira         987654321          14:28
   Ana Costa            456789123          14:15

↑/↓ Navegar             ← Rejeitar / excluir da fila
→ Aprovar / acolher     Esc Voltar
```

A fila deverá trabalhar com paginação/rolagem quando houver mais pedidos que linhas disponíveis. O identificador Telegram é a chave de identidade; username e nome de exibição podem mudar e não devem ser suficientes para conceder acesso.

## Tela 3 — confirmação obrigatória

```text
┌─ Confirmação ───────────────────────────────────────────┐
│ Ação: APROVAR                                         │
│ Bot: bioma                                            │
│ Usuário: Maria Oliveira                               │
│ Telegram User ID: 123456789                           │
│                                                        │
│ Autorizar este usuário a utilizar o bot bioma?         │
│                                                        │
│ Pressione Y para confirmar; N ou Esc para cancelar.    │
└────────────────────────────────────────────────────────┘
```

A rejeição utiliza a mesma confirmação, mas com a ação **REJEITAR** e texto que explique que a solicitação sairá da fila, preservando histórico.

## Teclado

| Contexto | Tecla | Ação |
|---|---|---|
| Lista de bots ou pedidos | `↑` / `↓` | Mover seleção sem executar alterações |
| Lista de bots | `Enter` | Abrir bot selecionado |
| Lista de pedidos | `→` | Preparar aprovação do pedido selecionado |
| Lista de pedidos | `←` | Preparar rejeição/remoção da fila do pedido selecionado |
| Confirmação | `Y` ou `y` | Confirmar aprovação ou rejeição |
| Confirmação | `N`, `n` ou `Esc` | Cancelar sem enviar mutação |
| Lista | `Esc` | Voltar ao nível anterior |

Nenhuma alteração persistente acontece apenas com `←` ou `→`: **a confirmação com `Y` é obrigatória em ambas as ações**. Durante a confirmação, teclas de navegação não devem alterar a pessoa em análise.

## Persistência e autorização propostas

| Informação | Necessidade |
|---|---|
| Identidade | `client_id`, `telegram_user_id`, `chat_id`; display name/username somente informativos |
| Solicitação | Identificador único, estado `pending / approved / rejected`, datas de criação e decisão |
| Auditoria | Operador Linux que decidiu, tipo da decisão, momento e referência ao pedido |
| Autorização | Concessão/revogação por `client_id + telegram_user_id` e escopo de chat quando aplicável |
| Fila | Contagem, ordenação e paginação de pendências por cliente, sem misturar bots |

Os nomes acima indicam **conceitos de contrato**, não tabelas SQL ou campos aprovados. O modelo físico e o mecanismo de retomada de pedidos rejeitados continuam pendentes. Para não exigir restart na aprovação, será necessária uma fonte persistida de autorizações no PostgreSQL e sua reconciliação com o `allowed_users` atual do YAML.

## API UDS administrativa — capacidades necessárias

| Operação lógica | Dados de entrada | Retorno / falhas |
|---|---|---|
| Listar bots e pendências | Filtros e paginação opcionais | Clientes acessíveis, totais pendentes |
| Listar pedidos por bot | `client_id`, paginação/ordenação | Pedidos, identidade, horário e estado |
| Aprovar pedido | ID do pedido, `client_id`, identidade do operador e versão/estado esperado | Confirmação de aprovação; `not_authorized`, `already_decided`, `not_found` |
| Rejeitar pedido | ID do pedido, `client_id`, versão/estado esperado | Confirmação de rejeição; erros equivalentes |

Os nomes de operações, envelope, códigos e framing são **exemplos de comportamento a especificar**, não contratos implementados. O serviço usa a identidade confiável da conexão local, não uma string de “operador” fornecida pelo cliente como prova de permissão.

**Requisitos de segurança:** os sockets são acessíveis pelo grupo `absabia`, mas permitir conexão não deve bastar para aprovar usuários. O serviço precisa validar permissão administrativa por bot; comparar o estado esperado em transação para evitar decisões concorrentes; não usar nome exibido como identidade; registrar auditoria; impedir que duas aprovações/rejeições da mesma pendência produzam efeitos contraditórios.

## Situações de erro e apresentação

- **Nenhum pedido:** lista vazia com mensagem explícita, sem seleção fictícia.
- **Pedido decidido por outro operador:** informar “Solicitação já analisada” e atualizar a fila.
- **Bot desabilitado/inexistente:** não permitir decisão, informar o motivo.
- **Sem autorização do operador:** negar mutação, mantendo a lista.
- **Serviço/UDS/PostgreSQL indisponível:** mostrar falha, sem presumir aprovação e sem fazer acesso direto ao banco.
- **Muitos pedidos:** paginação ou rolagem, com número de pendências.
- **Terminal sem cores:** manter seleção e status legíveis com símbolos e palavras.

## Referências

- [Guia das telas](guia-usuario.md)
- [Interfaces e serviços UDS](interfaces-servicos.md)
- [Segurança e permissões](seguranca-acessos.md)
- [Persistência existente](persistencia.md)
- [Fluxo /start e contato — FLW-006](01-Project_Charter/scope/flows/FLW-006-start-registro-de-contato-e-apresentacao-do-menu.md)
- [Modelo de configuração legado — CFG-0001](especificacao/configuracao/modelo-configuracao.md)
- [Telegram Bot API (referência oficial)](https://core.telegram.org/bots/api)
