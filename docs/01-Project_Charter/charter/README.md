# Sabiá — Project Charter

![Document](https://img.shields.io/badge/ID-PCH--0001-0550ae?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

## Problema ou oportunidade

Aplicações, scripts e serviços locais do ecossistema Bioma precisam disponibilizar operações remotas e alertas sem expor shell arbitrário, sem exigir portas públicas para o canal inicial e sem acoplar regras operacionais diretamente ao Telegram.

O projeto também precisa manter operações demoradas, mensagens pendentes, alertas e mídias recuperáveis diante de reinicializações do serviço.

Além disso, contatos conhecidos e o catálogo de ações oferecido aos usuários precisam sobreviver a reinicializações e poder ser administrados sem recompilar o serviço.

## Propósito

O Sabiá existe para atuar como uma ponte controlada entre usuários autorizados, aplicações locais do ecossistema Bioma e o Telegram, oferecendo execução de operações previamente cadastradas, processamento assíncrono, monitoramento, alertas e transporte de mídia com autorização explícita e persistência operacional.

O serviço também mantém contatos conhecidos e um catálogo persistente de ações, administrável localmente pelo próprio executável do Sabiá, para que menus e operações possam evoluir sem recompilação.

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

## Objetivos

- [OBJ-001 — Permitir que usuários autorizados acionem operações locais previamente cadastradas por meio dos clientes Telegram do Sabiá.](objectives/OBJ-001-acionar-operacoes-locais-cadastradas.md)
- [OBJ-002 — Manter operações demoradas fora do tratamento síncrono de comandos.](objectives/OBJ-002-processamento-assincrono.md)
- [OBJ-003 — Executar verificações agendadas e produzir alertas relevantes orientados a mudança de estado.](objectives/OBJ-003-verificacoes-agendadas-e-alertas.md)
- [OBJ-004 — Permitir ingestão local e posterior entrega de mídia sem exigir que produtores conheçam detalhes da Telegram Bot API.](objectives/OBJ-004-ingestao-e-entrega-de-midia.md)
- [OBJ-005 — Preservar estado operacional necessário para recuperação após reinício.](objectives/OBJ-005-preservar-estado-operacional.md)
- [OBJ-006 — Limitar o acesso remoto e local às identidades e operações explicitamente autorizadas.](objectives/OBJ-006-restringir-acesso-e-execucao.md)
- [OBJ-007 — Registrar e manter o contato de usuários que iniciem relacionamento com um cliente Telegram do Sabiá.](objectives/OBJ-007-registrar-contatos-telegram.md)
- [OBJ-008 — Permitir que o operador administre um catálogo persistente e hierárquico de ações pelo terminal.](objectives/OBJ-008-administrar-catalogo-de-acoes.md)

## Não objetivos

Ver [NON-OBJECTIVES.md](NON-OBJECTIVES.md).

## Restrições

- [CON-001 — O serviço opera em Linux e o executável principal é distribuído como um único binário gerenciado pelo systemd.](constraints/CON-001-o-servico-opera-em-linux-e-o-executavel-principal-e-distribu.md)
- [CON-002 — O primeiro canal Telegram recebe updates por long polling e não depende de webhook público.](constraints/CON-002-o-primeiro-canal-telegram-recebe-updates-por-long-polling-e-.md)
- [CON-003 — O processo principal não executa como root e utiliza identidade Linux dedicada.](constraints/CON-003-o-processo-principal-nao-executa-como-root-e-utiliza-identid.md)
- [CON-004 — Operações remotas somente podem resolver identificadores previamente cadastrados; execução arbitrária de shell é proibida.](constraints/CON-004-operacoes-remotas-somente-podem-resolver-identificadores-pre.md)
- [CON-005 — PostgreSQL é a persistência oficial do estado operacional, sem fallback normativo para SQLite.](constraints/CON-005-postgresql-e-a-persistencia-oficial-do-estado-operacional-se.md)
- [CON-006 — Cada cliente Telegram mantém identidade, autorização e catálogo próprios.](constraints/CON-006-cada-cliente-telegram-mantem-identidade-autorizacao-e-catalo.md)
- [CON-007 — Interfaces locais compartilhadas com aplicações são limitadas às identidades Linux explicitamente autorizadas.](constraints/CON-007-interfaces-locais-compartilhadas-com-aplicacoes-sao-limitada.md)
- [CON-008 — Contatos e catálogo de ações são persistidos no PostgreSQL; o catálogo operacional não depende exclusivamente de configuração estática em arquivo.](constraints/CON-008-contatos-e-catalogo-de-acoes-sao-persistidos-no-postgresql-o.md)
- [CON-009 — Cadastro ou alteração de script/processador associado a uma ação ocorre somente por interface administrativa local do Sabiá, nunca por comando remoto do usuário.](constraints/CON-009-cadastro-ou-alteracao-de-script-processador-associado-a-uma-.md)
- [CON-010 — Limites de ingestão de mídia](constraints/CON-010-limites-de-ingestao-de-midia.md)

## Premissas

- [ASM-001 — A instalação possui conectividade de saída suficiente para acessar a Telegram Bot API.](assumptions/ASM-001-a-instalacao-possui-conectividade-de-saida-suficiente-para-a.md)
- [ASM-002 — Tokens e demais credenciais válidas serão fornecidos por mecanismo seguro de configuração.](assumptions/ASM-002-tokens-e-demais-credenciais-validas-serao-fornecidos-por-mec.md)
- [ASM-003 — Operações disponíveis aos usuários serão cadastradas previamente pelo operador pela interface administrativa local e persistidas no catálogo de ações.](assumptions/ASM-003-operacoes-disponiveis-aos-usuarios-serao-cadastradas-previam.md)
- [ASM-004 — Uma instância PostgreSQL compatível estará disponível ao serviço.](assumptions/ASM-004-uma-instancia-postgresql-compativel-estara-disponivel-ao-ser.md)
- [ASM-005 — Aplicações locais que utilizem interfaces compartilhadas serão executadas sob identidades autorizadas pelo ambiente Linux.](assumptions/ASM-005-aplicacoes-locais-que-utilizem-interfaces-compartilhadas-ser.md)

## Questões abertas

- [OPEN-001 — Autoridade formal de aprovação](open-questions/OPEN-001-autoridade-formal-de-aprovacao.md)

## Scope

Ver [../scope/README.md](../scope/README.md).
