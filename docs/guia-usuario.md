# Guia do usuário — `sabiactl` e Telegram

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

### Gerenciar menus, configurações e scripts

```text
SABIÁ > Administração

1. Listar e selecionar cliente
2. Listar/editar menu e ações
3. Consultar/alterar configuração permitida
4. Enviar script ao serviço por UDS
5. Verificar validação e solicitar ativação
0. Voltar
```

O operador seleciona a opção, preenche os campos necessários e confirma. O Sabiá valida a operação e devolve sucesso/erro estruturado; a aplicação mostra mensagens legíveis com cores para ajudar a distinguir estados. Upload de script não significa ativação nem execução automática.

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

- [Contrato operacional Telegram e exemplos de botões](telegram-operacao-2026.md)
- [Fluxo de `/start` e menu — FLW-006](01-Project_Charter/scope/flows/FLW-006-start-registro-de-contato-e-apresentacao-do-menu.md)
- [Cadastro de ações e execução — FLW-007](01-Project_Charter/scope/flows/FLW-007-cadastro-administrativo-e-execucao-de-acao-persistente.md)
- [Escopo do MVP](especificacao/requisitos/mvp.md)
- [Pendências dos contratos operacionais](pendencias.md)
