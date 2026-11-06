<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Criar um bot do Telegram para o Sabiá

> **Conteúdo lido e migrado para:** [GUIA-002 — documento canônico](guias/GUIA-002-criar-bot-telegram.md). O texto histórico permanece disponível para rastreabilidade; especificações em aberto não foram aprovadas.

Este guia mostra como **criar um bot no Telegram** e preparar suas informações para futura integração com o **A-Bioma Sabiá**. Criar o bot no Telegram já é possível; a integração automática com o Sabiá **ainda não está implementada** na branch `develop`.

## 1. Abrir o BotFather

1. Instale e abra o Telegram, usando uma conta própria.
2. Acesse o bot oficial [@BotFather](https://t.me/BotFather) e inicie a conversa.
3. Envie o comando `/newbot`.

O BotFather é o serviço oficial do Telegram para criar e administrar bots.

## 2. Escolher nome e identificador

O BotFather solicitará duas informações:

| Campo | Exemplo | Regra |
|---|---|---|
| Nome visível | `A-Bioma Sabiá` | Nome exibido no perfil do bot |
| Username | `ABiomaSabiaBot` | Deve estar disponível e ser exclusivo |

O username deve ter **de 5 a 32 caracteres**, conter letras latinas, números ou sublinhado (`_`) e terminar com `bot` (sem diferenciação entre maiúsculas e minúsculas na disponibilidade). Se o nome estiver ocupado, escolha outro. Os exemplos deste guia **não têm disponibilidade verificada**.

> **Reserva de nome:** ao criar o bot, o username escolhido passa a identificá-lo. Não existe neste procedimento uma reserva separada de username sem criar um bot.

## 3. Guardar o token de acesso

Ao concluir o cadastro, o BotFather entregará um **token de autenticação**. Trate-o como uma senha:

- **Não** cole o token em README, commits, issues, capturas de tela, logs ou mensagens compartilhadas.
- Guarde-o em um mecanismo de segredos ou ambiente protegido usado pelo serviço.
- Se ele vazar, gere um novo token no BotFather usando `/token` e atualize a credencial no servidor.
- Cada bot possui token próprio. Nunca reutilize o mesmo token como credencial para outros serviços.

**O token controla o bot**. O username e o link público não devem ser confundidos com esse segredo.

## 4. Abrir e compartilhar o link

Para um username `ABiomaSabiaBot`, o endereço será:

```text
https://t.me/ABiomaSabiaBot
```

Cole o link no navegador ou no Telegram para abrir a conversa. Pressione **Iniciar** ou envie `/start`.

**Importante:** a criação no BotFather não instala um programa nem conecta automaticamente o bot ao Sabiá. Sem um backend atendendo à Bot API, o bot pode não responder ao `/start`.

## 5. Personalizar o perfil e os comandos (opcional)

No [@BotFather](https://t.me/BotFather), envie `/mybots`, selecione o bot criado e ajuste nome, descrição, imagem e comandos. Também há comandos como:

| Comando BotFather | Finalidade |
|---|---|
| `/setdescription` | Descrição mostrada no início da conversa |
| `/setabouttext` | Texto curto do perfil |
| `/setuserpic` | Foto do bot |
| `/setcommands` | Lista de comandos sugeridos pelo Telegram |
| `/token` | Gerar novo token de autenticação |

Exemplo de lista de comandos para o bot **Bioma**, conforme o MVP **planejado**:

```text
start - Abrir o menu principal
help - Mostrar ajuda
status - Consultar o estado do serviço
cpu - Consultar uso de CPU
memory - Consultar uso de memória
disk - Consultar armazenamento
uptime - Consultar tempo de atividade
```

Os comandos cadastrados no BotFather são **sugestões da interface Telegram**, não criam funcionalidades por si só. Configure apenas o que a aplicação efetivamente suportar quando estiver implementada.

## 6. Preparar a integração com o Sabiá

A documentação do Sabiá prevê múltiplos clientes lógicos isolados, inicialmente `bioma`, `tools` e `alerts`. Você pode criar **um bot separado para cada cliente**, repetindo o procedimento:

| Cliente Sabiá | Username ilustrativo | Referência de segredo sugerida |
|---|---|---|
| `bioma` | `ABiomaSabiaBot` | `SABIA_BIOMA_TOKEN` |
| `tools` | `ABiomaToolsBot` | `SABIA_TOOLS_TOKEN` |
| `alerts` | `ABiomaAlertsBot` | `SABIA_ALERTS_TOKEN` |

**Os usernames não foram reservados ou verificados.** Os nomes das variáveis acima são sugestões de operação, não configuração implantada.

De acordo com a proposta [CFG-0001](especificacao/configuracao/modelo-configuracao.md), a configuração operacional fica em `/etc/sabia/sabia.yaml` e cada cliente Telegram usa `token_env` para referenciar uma variável de ambiente protegida. Um **trecho ilustrativo**, que não é um arquivo YAML completo e não deve ser utilizado como configuração validada, seria:

```yaml
telegram:
  clients:
    bioma:
      enabled: true
      token_env: SABIA_BIOMA_TOKEN
      # allowed_users: definido conforme o esquema final
      # allowed_chats: opcional
```

O formato definitivo de `telegram.clients` e dos campos de autorização ainda está em `refinement`. Não inserir o token real no YAML. A autorização de usuários Telegram é **diferente** da permissão Linux `absabia`, usada para as APIs UDS locais.

O canal remoto previsto é o **long polling** (`getUpdates`) da Telegram Bot API, por HTTPS de saída. **Não é necessário expor um webhook público** para esse desenho. Para integrar o bot, o serviço terá de implementar o processamento dos updates e o envio de respostas; essa etapa ainda está pendente.

## 7. Conferir o resultado

Antes de considerar a criação concluída, confirme:

- [ ] O BotFather confirmou o username desejado.
- [ ] Você consegue abrir `https://t.me/<username_do_bot>`.
- [ ] O token foi guardado fora do repositório.
- [ ] O bot correto está associado ao cliente lógico desejado (`bioma`, `tools` ou `alerts`), ao menos no planejamento da configuração.
- [ ] A lista de usuários/chats autorizados será definida antes de habilitar comandos restritos.

O bot existir no Telegram **não comprova** que o Sabiá está em execução ou que responde mensagens.

## Referências

- [Telegram — criação e administração com BotFather](https://core.telegram.org/bots/features)
- [Telegram — tutorial de criação e token](https://core.telegram.org/bots/tutorial)
- [Telegram Bot API — autenticação e `getUpdates`](https://core.telegram.org/bots/api)
- [Projeto — configuração planejada](especificacao/configuracao/modelo-configuracao.md)
- [Projeto — interfaces e serviços](interfaces-servicos.md)
- [Projeto — operação Telegram](telegram-operacao-2026.md)
