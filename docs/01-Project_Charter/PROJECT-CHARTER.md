# Sabiá — Project Charter

![Document](https://img.shields.io/badge/ID-PCH--0001-0550ae?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

## Identificação

| Campo | Valor |
|---|---|
| Projeto | Sabiá |
| Responsável | Não definido — ver OPEN-001 |
| Status | refinement |
| Version | - |

## Problema ou oportunidade

Aplicações, scripts e serviços locais do ecossistema Bioma precisam disponibilizar operações remotas e alertas sem expor shell arbitrário, sem exigir portas públicas para o canal inicial e sem acoplar regras operacionais diretamente ao Telegram.

O projeto também precisa manter operações demoradas, mensagens pendentes, alertas e mídias recuperáveis diante de reinicializações do serviço.

Além disso, contatos conhecidos e o catálogo de ações oferecido aos usuários precisam sobreviver a reinicializações e poder ser administrados sem recompilar o serviço.

## Propósito

O Sabiá existe para atuar como uma ponte controlada entre usuários autorizados, aplicações locais do ecossistema Bioma e o Telegram, oferecendo execução de operações previamente cadastradas, processamento assíncrono, monitoramento, alertas e transporte de mídia com autorização explícita e persistência operacional.

O serviço também mantém contatos conhecidos e um catálogo persistente de ações, administrável localmente pelo próprio executável do Sabiá, para que menus e operações possam evoluir sem recompilação.

## Objetivos

| ID | Objetivo | Critério verificável |
|---|---|---|
| OBJ-001 | Permitir que usuários autorizados acionem operações locais previamente cadastradas por meio dos clientes Telegram do Sabiá. | Um comando permitido percorre Telegram → Sabiá → operação cadastrada → resposta pelo mesmo cliente e destino. |
| OBJ-002 | Manter operações demoradas fora do tratamento síncrono de comandos. | Operações demoradas podem ser executadas como jobs sem impedir o recebimento de novos comandos e podem informar progresso e resultado. |
| OBJ-003 | Executar verificações agendadas e produzir alertas relevantes orientados a mudança de estado. | Uma transição relevante gera notificação e estados repetidos não geram alerta imediato, salvo lembrete configurado. |
| OBJ-004 | Permitir ingestão local e posterior entrega de mídia sem exigir que produtores conheçam detalhes da Telegram Bot API. | Um produtor local autorizado consegue associar mídia a uma requisição e o Sabiá consegue entregá-la ao destino correlacionado. |
| OBJ-005 | Preservar estado operacional necessário para recuperação após reinício. | Requisições, jobs, entregas, mídias e estados de alerta necessários à retomada permanecem identificáveis após restart. |
| OBJ-006 | Limitar o acesso remoto e local às identidades e operações explicitamente autorizadas. | Usuários não autorizados e operações não cadastradas não são executados; o serviço não requer execução como root. |
| OBJ-007 | Registrar e manter o contato de usuários que iniciem relacionamento com um cliente Telegram do Sabiá. | O comando `/start` cria ou atualiza um contato persistente com identidade do usuário, cliente e destino de comunicação. |
| OBJ-008 | Permitir que o operador administre um catálogo persistente e hierárquico de ações pelo terminal. | O executável `sabia` permite cadastrar e consultar uma ação lógica, incluindo seu caminho e, quando aplicável, o script/processador associado, sem recompilar o serviço. |

## Não objetivos

| ID | Não objetivo | Motivo |
|---|---|---|
| NOBJ-001 | Oferecer shell remoto arbitrário ou executar comandos/caminhos fornecidos livremente pelo usuário. | O projeto opera somente sobre operações previamente cadastradas. |
| NOBJ-002 | Expor webhook público para receber updates do Telegram no primeiro escopo operacional. | O canal inicial utiliza conexões de saída por long polling. |
| NOBJ-003 | Implementar processadores complexos de imagem, vídeo ou áudio no primeiro MVP. | O primeiro MVP entrega a infraestrutura para integração, jobs e transporte de mídia; processadores complexos são evolução posterior. |
| NOBJ-004 | Disponibilizar HTTP/REST, gRPC, TCP ou outros canais remotos como interface do primeiro MVP. | Esses canais são possibilidades de evolução e não pertencem à fronteira atual. |
| NOBJ-005 | Conceder a aplicações locais acesso direto ao banco de dados ou aos diretórios internos do Sabiá. | As integrações locais devem ocorrer somente pelas interfaces explicitamente autorizadas. |
| NOBJ-006 | Entregar, no primeiro MVP, o mecanismo completo de assinaturas criadas pelo usuário e mailings periódicos, como `notícias/naval`. | A estrutura é mantida como backlog para evolução posterior sobre contatos e ações persistentes. |

## Stakeholders

| Stakeholder | Interesse / responsabilidade |
|---|---|
| Usuário Telegram autorizado | Solicitar operações permitidas, acompanhar jobs e receber resultados ou alertas. |
| Operador do ambiente Sabiá | Instalar, configurar e administrar credenciais, clientes, permissões, catálogo de ações, scripts/processadores e schedules, inclusive pela CLI administrativa. |
| Aplicações e serviços locais do ecossistema Bioma | Produzir ou consumir operações e mídia por interfaces autorizadas. |
| Responsável formal pelo projeto | Aprovar propósito, limites e mudanças de escopo; ainda não identificado formalmente. |

## Responsabilidade resumida

O Sabiá recebe interações de usuários autorizados pelo Telegram e entradas locais explicitamente permitidas, registra contatos conhecidos, mantém um catálogo persistente de ações, aplica autorização e roteamento, aciona operações cadastradas, coordena jobs e verificações agendadas, mantém o estado operacional necessário e entrega mensagens ou mídias ao destino correspondente.

O próprio executável `sabia` oferece uma interface administrativa local para manutenção desse catálogo.

## Limites principais

- A responsabilidade remota começa quando um update é recebido por um cliente Telegram habilitado.
- A responsabilidade local começa quando uma aplicação autorizada utiliza uma interface local publicada pelo Sabiá ou quando uma verificação cadastrada é disparada pelo próprio scheduler.
- Operações originadas por usuários são limitadas a identificadores previamente cadastrados; texto recebido não se torna shell arbitrário.
- Caminhos de scripts, executáveis, interpretadores e demais definições de execução são cadastrados localmente pelo operador; o usuário remoto não pode defini-los ou substituí-los.
- A hierarquia do menu é derivada do caminho lógico persistido da ação, sem exigir uma estrutura paralela específica para representar a árvore.
- O Sabiá é responsável por correlacionar requisição, execução e destino de resposta enquanto a operação estiver sob seu controle.
- A responsabilidade pela entrega externa termina quando o serviço remoto aceita a mensagem ou mídia e o Sabiá registra o resultado correspondente.
- Sistemas, scripts e aplicações acionados permanecem responsáveis pela lógica específica de negócio ou processamento que executam.

## Sistemas relacionados

| Sistema | Relação em alto nível |
|---|---|
| Telegram Bot API | Canal externo inicial para receber comandos e entregar mensagens e mídias. |
| Linux / ecossistema Bioma | Ambiente onde o Sabiá executa e integra operações previamente cadastradas. |
| PostgreSQL | Persistência operacional necessária para contatos, catálogo de ações, filas, correlações, recuperação, mídia e estado de alertas. |
| systemd | Gerenciamento do ciclo de vida do serviço no ambiente Linux. |

## Critérios de sucesso

- Um usuário autorizado consegue executar uma operação cadastrada e receber o resultado pelo cliente Telegram correto.
- Operações demoradas não bloqueiam o recebimento de novos comandos.
- Verificações agendadas conseguem gerar alertas por mudança de estado e comunicar recuperação.
- O serviço não exige shell arbitrário, execução como root nem endpoint público de webhook para funcionar no primeiro MVP.
- Um produtor local autorizado consegue enviar mídia ao Sabiá e a entrega pendente pode sobreviver a reinicializações.
- Clientes Telegram mantêm autorização e catálogo próprios, sem compartilhamento implícito.
- `/start` registra ou atualiza um contato e permite apresentar as ações habilitadas para aquele contexto.
- O operador consegue cadastrar pelo terminal uma ação como `/acoes/videoytb` associada a um script permitido, e o usuário consegue acioná-la sem fornecer o caminho do executável.
- Uma ação assíncrona pode produzir progresso, resultado textual e arquivos correlacionados ao solicitante.

## Restrições

| ID | Restrição | Origem |
|---|---|---|
| CON-001 | O serviço opera em Linux e o executável principal é distribuído como um único binário gerenciado pelo systemd. | Decisão vigente do projeto. |
| CON-002 | O primeiro canal Telegram recebe updates por long polling e não depende de webhook público. | Decisão vigente do projeto. |
| CON-003 | O processo principal não executa como root e utiliza identidade Linux dedicada. | Requisito de segurança vigente. |
| CON-004 | Operações remotas somente podem resolver identificadores previamente cadastrados; execução arbitrária de shell é proibida. | Requisito de segurança vigente. |
| CON-005 | PostgreSQL é a persistência oficial do estado operacional, sem fallback normativo para SQLite. | Decisão vigente do projeto. |
| CON-006 | Cada cliente Telegram mantém identidade, autorização e catálogo próprios. | Decisão vigente do projeto. |
| CON-007 | Interfaces locais compartilhadas com aplicações são limitadas às identidades Linux explicitamente autorizadas. | Requisito de segurança vigente. |
| CON-008 | Contatos e catálogo de ações são persistidos no PostgreSQL; o catálogo operacional não depende exclusivamente de configuração estática em arquivo. | Decisão de escopo vigente. |
| CON-009 | Cadastro ou alteração de script/processador associado a uma ação ocorre somente por interface administrativa local do Sabiá, nunca por comando remoto do usuário. | Requisito de segurança e administração vigente. |

## Premissas

| ID | Premissa | Precisa de Discovery? |
|---|---|---|
| ASM-001 | A instalação possui conectividade de saída suficiente para acessar a Telegram Bot API. | não |
| ASM-002 | Tokens e demais credenciais válidas serão fornecidos por mecanismo seguro de configuração. | não |
| ASM-003 | Operações disponíveis aos usuários serão cadastradas previamente pelo operador pela interface administrativa local e persistidas no catálogo de ações. | não |
| ASM-004 | Uma instância PostgreSQL compatível estará disponível ao serviço. | não |
| ASM-005 | Aplicações locais que utilizem interfaces compartilhadas serão executadas sob identidades autorizadas pelo ambiente Linux. | não |

## Questões abertas

| ID | Questão | Bloqueadora? | Discovery |
|---|---|---|---|
| OPEN-001 | Quem é o responsável formal por aprovar o Charter, o Scope e futuras mudanças de escopo do Sabiá? | não | - |

## Critério de fechamento

O Charter só pode atingir `refined` quando propósito, objetivos, não objetivos, limites, critérios de sucesso, restrições e premissas críticas estiverem suficientemente definidos e as questões abertas que afetem a fronteira do sistema estiverem resolvidas.
