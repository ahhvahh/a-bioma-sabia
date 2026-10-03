# Sumário Executivo

Este documento descreve em detalhes os passos e requisitos necessários para integrar e testar o bot **Sabiá** com a API de Bots do Telegram, baseando-se na [documentação oficial do Telegram Bot API](https://core.telegram.org/bots/api). Incluímos instruções sobre registro do bot, declaração de comandos, formatos de mensagens, entrega de updates (getUpdates vs webhook), métodos de envio de mensagens/mídia, modelos de conversas, segurança e limites de taxa. Também apresentamos exemplos de payloads JSON para o comando `/vidconvert` e propomos um **agente fake de Telegram** (repositório scaffold) para testes locais, incluindo fluxos em mermaid, tabelas de exemplo e um esboço de README. Toda a informação técnica é suportada por trechos da documentação oficial (citados) e explicações em português.

## 1. Registrar o Bot

Para criar um novo bot, utiliza-se o **@BotFather** no Telegram. Envie `/newbot` e siga os passos para nomear o bot e escolher um nome de usuário. Ao final, o BotFather retorna um *token* único (como uma “senha”) para acesso à API do bot. É fundamental guardar este token em segurança. 

- **Token:** Serve como credencial (ex: `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`). 
- **Políticas e Limites:** Respeite as regras do Telegram (ex.: não exceder 30 mensagens/segundo sem notificar, conforme seção *Paid Broadcasts*). 
- **Modo de Acesso:** Por padrão, todo bot opera em “modo de privacidade”: só recebe mensagens se o usuário iniciar a conversa (com `/start`) ou interagir explicitamente (por exemplo, adicionando o bot a um grupo ou usando menções). 

Em resumo, o fluxo básico de registro é: contatar o BotFather, criar o bot e obter o token. Após isso, configure seu servidor para usar esse token nas chamadas à API.

## 2. Declarar/Registrar Comandos

Para mostrar comandos no menu do bot, há duas abordagens:

- **Via BotFather:** Abra o chat com **@BotFather**, selecione **/mybots > SeuBot > Edit Bot > Edit Commands**, e adicione os comandos desejados com suas descrições. Exemplo: digitar `/mybots`, escolher o bot, clicar em *Edit Commands* e inserir:
  
  ```
  /vidconvert - Converte vídeo do YouTube para envio no chat
  /status - Mostra o status de conversão atual
  ```

- **Via API (programaticamente):** Use o método [**setMyCommands**] do Bot API para definir até 100 comandos de uma vez. Por exemplo, enviar JSON `{"commands":[{"command":"vidconvert","description":"Converte vídeo do YouTube"},{"command":"status","description":"Ver status"}]}` via HTTPS. Esse método suporta *escopos* (p. ex. comandos específicos para grupos ou idiomas) usando `BotCommandScope`. Há também `getMyCommands` e `deleteMyCommands` para manipular via API.

Além dos comandos, configure em BotFather descrições do bot (*/setdescription*), nome curto, foto de perfil, etc. Você pode definir um botão de menu padrão com `setChatMenuButton`. Tudo isso é documentado na API.

## 3. Formato de Updates e Mensagens

O Telegram envia eventos ao bot como objetos **Update** em JSON. Cada `Update` contém um `update_id` sequencial (útil para evitar duplicação) e um dos campos opcionais, como `message`, `edited_message`, `callback_query`, `inline_query`, etc. Apenas um desses campos estará presente por update. Por exemplo, um update típico de texto contém:

```json
{
  "update_id": 12345678,
  "message": {
    "message_id": 99,
    "from": {"id": 111222333, "is_bot": false, "first_name": "Alice", "username": "alice", "language_code": "pt"},
    "chat": {"id": 111222333, "first_name": "Alice", "username": "alice", "type": "private"},
    "date": 1696340000,
    "text": "/vidconvert https://www.youtube.com/watch?v=XYZ",
    "entities": [{"offset": 0, "length": 10, "type": "bot_command"}]
  }
}
```

Neste exemplo, `update_id` é único para o update. O campo `message` é um objeto **Message**. Ele inclui `text` (mensagem em UTF-8) e `entities` (lista de entidades no texto, como comandos ou links). O objeto `chat` informa `chat.id` e `chat.type` (por exemplo “private”, “group”, “supergroup” ou “channel”). Outros campos úteis em `Message` incluem `reply_to_message` (para respostas), e campos de mídia como `photo` (array de **PhotoSize**), `video`, `document`, etc. 

Além de mensagens de texto, o update pode vir via `callback_query` (quando o usuário aperta botão inline) ou `inline_query` (quando o usuário usa seu bot em modo inline). Por exemplo, um **CallbackQuery** possui `id`, `from` (usuário), e `data` (string do botão). Um **InlineQuery** contém o campo `query` (texto digitado pelo usuário). Para cada tipo de update, use o campo apropriado no JSON.

## 4. Entrega de Updates (long polling vs webhook)

O bot pode receber updates de duas formas:

- **getUpdates (long polling):** O bot chama periodicamente `getUpdates` via HTTPS. Cada chamada retorna uma lista de updates novos (JSON). Use o parâmetro `offset` para confirmar updates já processados (setar `offset` como o maior `update_id` recebido +1), evitando duplicatas. É recomendável usar `timeout` para long polling (ex: 60s) para eficiência. Atenção: enquanto o webhook estiver ativo, `getUpdates` não funciona. 

- **setWebhook (webhook HTTPS):** Configure `setWebhook` com uma URL segura HTTPS. Toda vez que houver update, o Telegram fará um POST para essa URL contendo o JSON do `Update`. O bot deve validar `X-Telegram-Bot-Api-Secret-Token` se configurado, para garantir origem. Se o POST falhar (status HTTP != 2xx), o Telegram re-tentará várias vezes. Parâmetros úteis incluem `allowed_updates` (para filtrar tipos de update) e `drop_pending_updates` (para apagar fila prévia). No webhook, é necessário fornecer certificado SSL válido (ou fazer upload do certificado público se for auto-assinado). Webhooks suportam portas 443, 80, 88, 8443.

Em ambos os casos, trate cada update de forma idempotente: use `update_id` para não processar duas vezes o mesmo evento. Após processar um update em long polling, ajuste o offset. Em webhooks, para duplicação basta rastrear pelo `update_id` e ignorar repeats.

## 5. Enviando Mensagens de Volta

Para responder aos usuários, use os métodos de envio da API. O básico é o [`sendMessage`](https://core.telegram.org/bots/api#sendmessage), que envia texto puro. Os parâmetros principais são `chat_id` (o ID do chat ou username de canal alvo) e `text`. É possível definir `parse_mode` (por exemplo "MarkdownV2" ou "HTML") para formatação, bem como `reply_markup` para teclados inline ou customizados. Exemplo simples:
```http
POST https://api.telegram.org/bot<token>/sendMessage
Content-Type: application/json

{"chat_id":12345678, "text":"Texto de resposta", "parse_mode":"MarkdownV2"}
```
Além de texto, existem métodos específicos para mídia:

- **sendPhoto:** Envia imagem (JPEG). Parâmetros: `photo` (InputFile ou URL ou `file_id`) e `caption` (legenda). O arquivo deve ter no máximo 10MB e razão de aspecto não maior que 20.

- **sendVideo:** Envia vídeo (MPEG4). Parâmetros: `video` (InputFile/URL/file_id), `caption`, `thumbnail` (opcional). Tamanho máximo 50MB atualmente.

- **sendDocument:** Envia arquivo genérico. Parâmetros: `document`, `caption`. Suporta até 50MB de qualquer tipo de arquivo.

- **sendMediaGroup:** Envia um álbum (2 a 10 mídias). Use um array de objetos `InputMedia` (como fotos, vídeos, documentos). Retorna várias mensagens.

Em todos eles, inclua `chat_id`. Você também pode usar `reply_to_message_id` para responder a uma mensagem específica, e `disable_notification` para envio silencioso (sem som). Use `reply_markup` (inline keyboards) para botões no chat. Consulte a documentação para cada método na seção de **Bot API** (ex: *sendPhoto*, *sendVideo* etc.) para ver todos os parâmetros disponíveis.  

## 6. Envio/Recebimento de Arquivos e Mídia

Quando o usuário envia arquivos (fotos, vídeos, documentos) ao bot, o Update traz esses arquivos em campos específicos de `message`:

- **Fotos:** O campo `photo` é um array de objetos [PhotoSize] (com `file_id` para cada tamanho). Use o `file_id` desejado para baixar ou reusar. 
- **Vídeos:** O campo `video` traz objeto [Video] com `file_id`.
- **Documentos:** O campo `document` traz objeto [Document] com `file_id`.
- **Outros:** Há `voice` (mensagem de voz), `animation` (GIF) etc.

Para **baixar** um arquivo enviado, chame o método [`getFile`](https://core.telegram.org/bots/api#getfile) passando o `file_id`. Ele retorna um objeto `File` com `file_path`. Você pode então baixar de `https://api.telegram.org/file/bot<token>/<file_path>` (link válido por pelo menos 1 hora). Atenção: bots podem baixar até 20MB por vez. Salve localmente o conteúdo ou registre o link no seu DB para posterior acesso. O nome original e MIME podem não ser preservados; considere armazenar `file_path`, `file_size`, `mime_type` (quando disponíveis em Document).

Para **enviar** arquivos, você tem três opções:

1. **Reusar um file_id:** Se o arquivo já foi enviado anteriormente pelo bot, basta fornecer o mesmo `file_id` (mais eficiente).
2. **URL pública:** Informe uma URL HTTP como string; o Telegram fará o download direto.
3. **Upload multipart:** Envie um novo arquivo diretamente no request multipart/form-data (usando `InputFile`). Isto é necessário para arquivos locais. Exemplo (Node.js):
   ```js
   bot.sendVideo(chatId, fs.createReadStream("/path/video.mp4"));
   ```
   Os tamanhos são limitados: fotos ≤10MB, documentos/áudios/vídeos ≤50MB. Respeite esses limites para evitar falhas. O tipo de conteúdo (`content_type`) do multipart deve corresponder (ex. `video/mp4`). 

O Telegram lida com streaming/chunking internamente. Basta enviar o arquivo inteiro ou stream e a biblioteca do bot cuidará da multipart/form. Depois, Telegram retornará um objeto Message contendo `file_id`, `file_size`, etc. Você pode então usá-los em mensagens futuras.

## 7. Modelagem de Conversas/Sessões

O Telegram identifica conversas principalmente pelo **chat**. Cada mensagem em `Update` inclui `message.chat`, um objeto Chat. Os tipos possíveis são: 

- **private:** conversa privada bot–usuário (chat.id é o usuário). 
- **group/supergroup:** grupos de chat (chat.id comum ao grupo). 
- **channel:** canal (bot deve ser administrador para receber posts).

Para bots em **supergrupos com tópicos**, há o campo `message_thread_id` (ID do tópico no fórum). Para cada mensagem, se for resposta a outra (`reply_to_message`), esse objeto conterá a mensagem original. Isto permite filtrar por contexto ou determinar a quem responder.

**Limitações:** O Telegram não mantém estado de sessão além do chat. Informações de “etapa da conversa” (por exemplo, se o usuário já iniciou um processo ou em que etapa está) **precisam ser armazenadas pelo seu sistema** (banco de dados). O bot só recebe as mensagens e contexto imediato (IDs, texto, entidades). Assim, gerencie o estado da aplicação (por exemplo, mapeando chat_id para registro de usuário) no seu backend. 

## 8. Segurança e Rate Limits

Algumas práticas importantes:

- **Segurança Webhook:** Use HTTPS com certificado válido. Você pode definir um `secret_token` em `setWebhook` e verificar o cabeçalho `X-Telegram-Bot-Api-Secret-Token` nas requests para confirmar que vem do Telegram. Certifique-se de remover webhook (`deleteWebhook`) se voltar a usar polling.
- **Limites de envio:** Por padrão, um bot pode enviar até **30 mensagens por segundo** em broadcast. Para campanhas maiores, existe a opção paga (Paid Broadcasts) para até 1000 msg/s. Fora isso, sempre verifique erros HTTP 429 (Too Many Requests). 
- **Privacidade do usuário:** Um bot **não pode** iniciar conversa com um usuário do zero (precisa primeiro receber uma mensagem dele ou ter deep linking `/start`). Respeite a **política anti-spam** do Telegram – por exemplo, não envie mensagens irrelevantes. 
- **Deduplicação:** Sempre use `update_id` para garantir que mesmo em falhas de conexão você não processe um mesmo update duas vezes.
- **Backoff e retries:** Ao usar webhooks, o Telegram faz vários retries automáticos em caso de falha. No polling, implemente lógica de re-tentativa em caso de erro de rede, mas cuidado para não duplicar updates já confirmados pelo offset.
- **Allowed Updates:** Configure `allowed_updates` em `setWebhook` para receber apenas os tipos necessários. Isso pode reduzir tráfego e exposição a eventos irrelevantes.

## 9. Exemplos de Payloads para `/vidconvert`

Para ilustrar, considere o comando `/vidconvert <link>`. Abaixo listamos exemplos de payloads e registros que podem ser usados no banco de dados.

- **Update JSON de entrada:** Exemplo de `Update` enviado pelo Telegram quando o usuário manda `/vidconvert https://youtube.com/...`. Note o campo `text` com o comando e as entidades indicando `bot_command`.

  ```json
  {
    "update_id": 2001,
    "message": {
      "message_id": 501,
      "from": {"id": 12345, "is_bot": false, "first_name": "Carlos", "username": "carlos", "language_code": "pt"},
      "chat": {"id": 12345, "first_name": "Carlos", "username": "carlos", "type": "private"},
      "date": 1696345000,
      "text": "/vidconvert https://youtu.be/dQw4w9WgXcQ",
      "entities": [{"offset": 0, "length": 11, "type": "bot_command"}]
    }
  }
  ```
  *Neste exemplo, `update_id`=2001 e `message.text` é o comando enviado pelo usuário.*  

- **Tabela de `requests` (exemplo):** Ao receber esse update, seu sistema pode criar uma entrada na tabela de requisições. Por exemplo:

  | request_id | user_id | chat_id | command      | argument                          | created_at          | status   |
  |-----------:|--------:|--------:|--------------|-----------------------------------|---------------------|---------|
  | 1          | 12345   | 12345   | "/vidconvert"| "https://youtu.be/dQw4w9WgXcQ"    | 2026-10-03 12:30:00 | PENDING |

- **Tabela de `jobs` (exemplo):** A seguir, cria-se um job de processamento associado à requisição:

  | job_id | request_id | status    | created_at          | started_at         | finished_at        | output_file      |
  |-------:|-----------:|-----------|---------------------|--------------------|--------------------|------------------|
  | 10     | 1          | QUEUED    | 2026-10-03 12:30:01 | (null)             | (null)             | (null)           |

- **Tabela de `media_transmissions` (exemplo):** Depois que o job é concluído e um arquivo de mídia (ex: MP4) é enviado, registre assim:

  | id | job_id | file_id                                | file_url                                    | media_type | status  | created_at          |
  |--:|-------:|----------------------------------------|---------------------------------------------|-----------|---------|---------------------|
  | 5  | 10     | "BQADBAADApYAAgc..."                    | "file/bot<token>/ABC123.mp4"                | "video"    | SENT    | 2026-10-03 12:45:00 |

Nos exemplos acima, usamos valores fictícios. A estrutura exata (nomes de colunas) depende do seu modelo de dados, mas deve cobrir os campos principais de usuário (`user_id`), chat (`chat_id`), comando original, e dados dos arquivos processados. Use *mermaid sequence diagrams* (abaixo) para visualizar os fluxos.

## 10. Agente Fake para Testes

Para testar localmente sem depender do Telegram real, recomendamos criar um **agente fake** (servidor simulado) com os seguintes componentes:

- **Arquitetura:** Um servidor simples (Node.js/Express ou Python/Flask) que simula endpoints da API do Telegram. 
- **Endpoints a implementar:** 
  - `/setWebhook` (simula resposta do Telegram definindo webhook)
  - `/getUpdates` (simula retornos de atualizações. Por exemplo, pode ler JSONs de teste e devolver ao bot)
  - `/fakeTelegram/file/<path>` (simula o servidor de arquivos do Telegram, servindo arquivos estáticos)
  - `/sendMessage`, `/sendVideo` etc. (para receber chamadas do bot fake e registrar o comportamento)
- **Simulação de webhook:** O bot em teste deve apontar para o agente fake (URL local). O agente fake deve “soltar” updates simulados para o bot, via webhook ou retornos de getUpdates.
- **Scripts de teste:** Implemente scripts que:
  1. **Sincronizar cursor:** chamar `/getUpdates` até não haver novos dados (offset até último update).
  2. **Enviar comando:** enviar um update JSON de `/vidconvert` (via getUpdates ou webhook).
  3. **Enviar múltiplas mensagens do mesmo usuário:** simular user falando várias vezes, para testar fila e deduplicação.
  4. **Enviar arquivo de vídeo:** simular update com `message.video` e `file_id`, para testar download via `getFile`.
- **Contratos de mensagens:** Especifique como campos do JSON (Telegram) mapeiam para suas tabelas. Ex.: `Update.message.from.id → requests.user_id`, `text → requests.command+argumentos`.
- **Checklist de cenários:** Certifique-se de validar no teste fake:
  - *Registro do usuário:* `/start` cria usuário na base.
  - *Deduplicação:* o mesmo `update_id` não cria entradas duplicadas.
  - *Criação de Request:* comando `/vidconvert` gera uma linha em `requests`.
  - *Criação de Job:* imediatamente após, há um `job` ligado à request.
  - *Worker Claim:* um processo ou thread simulado busca jobs pendentes e atualiza status.
  - *Envio de status:* o bot envia mensagens intermediárias (ex: “Conversão iniciada”, “processando…”).
  - *Envio de arquivo:* ao final, o bot envia o vídeo via `sendVideo` e registra em `media_transmissions`.
  - *Entregas via Telegram:* verifique se o output do bot fake (`sendVideo`) é esperado para o usuário.
- **Tabelas de exemplo:** Inclua, como acima, exemplos de entradas nas tabelas para cada estágio (request, job, media).
- **Diagramas Mermaid:** Visualize fluxos importantes. Exemplo de diagrama de requisição a job:

  ```mermaid
  sequenceDiagram
    participant Usuário
    participant Bot
    participant DB
    participant Worker
    Usuário->>Bot: /vidconvert <link>
    Bot->>DB: CREATE request
    Bot->>DB: CREATE job (status=QUEUED)
    Bot-->>Usuário: "Pedido recebido, aguardando processamento"
    Worker->>DB: CLAIM job
    Worker-->>DB: UPDATE job (status=PROCESSING)
    loop Processa vídeo
      Worker-->>DB: UPDATE job (log/status progress)
    end
    Worker-->>DB: UPDATE job (status=DONE, output file)
    Bot-->>Usuário: sendChatAction(upload_video)
    Bot->>Bot: getFile(output file_id)
    Bot->>Usuário: sendVideo(file_id)
    Bot-->>DB: INSERT media_transmission
    ```
  
  Outro diagrama simplificado de **registro**:

  ```mermaid
  sequenceDiagram
    participant Usuário
    participant Bot
    participant DB
    Usuário->>Bot: /start
    Bot->>DB: CREATE new user (caso não exista)
    Bot-->>Usuário: "Olá, bem-vindo ao Sabiá!"
  ```

- **README.md (esboço):** No repositório do agente fake, inclua um README com instruções para rodar localmente. Exemplo:

  ```markdown
  # Fake Telegram Agent

  Este serviço simula os endpoints do Telegram para testar o bot **Sabiá** em ambiente local.

  ## Arquitetura
  - Servidor Node.js (Express) ou Python (Flask) com endpoints:
    - `/setWebhook`, `/deleteWebhook`
    - `/getUpdates`
    - `/file/{file_path}`
    - `/bot<token>/sendMessage`, `/sendVideo`, etc. (para registrar envios do bot)
  - Banco de dados simples (ou arrays em memória) para armazenar updates simulados.

  ## Executando localmente
  1. Clone o repositório:
     ```bash
     git clone https://example.com/fake-telegram-agent.git
     cd fake-telegram-agent
     ```
  2. Ajuste o arquivo `config.json` com o token de teste (igual ao do bot em teste).
  3. Configure o bot Sabiá para usar este servidor como webhook:
     - `https://<IP_LOCAL>:<PORT>/bot<token>/`
  4. Inicie o serviço:
     ```bash
     npm install
     npm start
     ```
     ou, se usar Python/Flask:
     ```bash
     pip install -r requirements.txt
     python app.py
     ```
  5. Use os scripts de teste para enviar comandos:
     ```bash
     ./scripts/send_vidconvert.sh https://youtu.be/example
     ./scripts/send_multiple.sh
     ./scripts/send_video_file.sh path/to/video.mp4
     ```
  6. Verifique os logs para ver as respostas do bot e atualizações nas tabelas de teste.

  ## Testes
  - **Teste de sincronização:** `scripts/sync_updates.sh` limpa o backlog de updates simulados.
  - **Envio de comando:** `scripts/send_command.sh` envia `/vidconvert`.
  - **Envio de multimídia:** `scripts/send_media.sh` simula o usuário enviando vídeo ou foto.
  - **Verificação de filas:** confere se os jobs são criados e processados.

  ## Tecnologia
  - Node.js v18+ ou Python 3.9+
  - Docker Compose opcional: um `docker-compose.yml` mínimo pode incluir o fake server e, se necessário, um DB SQLite/Postgres para simular armazenamento.

  ```
  
  Esse README ajuda qualquer desenvolvedor a executar o agente fake e validar o comportamento do bot sem acesso à infraestrutura real do Telegram.

> **Fontes:** Informações baseadas principalmente na [documentação oficial do Bot API](https://core.telegram.org/bots/api) do Telegram. Trechos citados foram traduzidos/adaptados para português para maior clareza. Para detalhes adicionais, consulte os links oficiais indicados em cada seção.

