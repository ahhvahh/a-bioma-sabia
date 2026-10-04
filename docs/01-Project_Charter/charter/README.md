# Sabiá — Project Charter

![Document](https://img.shields.io/badge/ID-PCH--0001-0550ae?style=flat-square)
![Status](https://img.shields.io/badge/Status-refinement-d4a72c?style=flat-square)
![Version](https://img.shields.io/badge/Version----6e7781?style=flat-square)

## Problema ou oportunidade

Aplicações, scripts e serviços locais do ecossistema Bioma precisam disponibilizar operações remotas e alertas sem expor shell arbitrário, sem exigir portas públicas para o canal inicial e sem acoplar regras operacionais diretamente ao Telegram.

O projeto também precisa manter operações demoradas, mensagens pendentes, alertas e mídias recuperáveis diante de reinicializações do serviço. Contatos conhecidos e o catálogo de ações oferecido aos usuários precisam sobreviver a reinicializações e poder ser administrados sem recompilar o serviço.

## Propósito

O Sabiá existe para atuar como uma ponte controlada entre usuários autorizados, aplicações locais do ecossistema Bioma e o Telegram, oferecendo execução de operações previamente cadastradas, processamento assíncrono, monitoramento, alertas e transporte de mídia com autorização explícita e persistência operacional.

O serviço também mantém contatos conhecidos e um catálogo persistente de ações, administrável localmente pelo próprio executável do Sabiá.

## Stakeholders

| Stakeholder | Interesse / responsabilidade |
|---|---|
| Usuário Telegram autorizado | Solicitar operações permitidas, acompanhar jobs e receber resultados ou alertas. |
| Operador do ambiente Sabiá | Instalar, configurar e administrar credenciais, clientes, permissões, catálogo de ações, scripts/processadores e schedules. |
| Aplicações e serviços locais do ecossistema Bioma | Produzir ou consumir operações e mídia por interfaces autorizadas. |
| Responsável formal pelo projeto | Aprovar propósito, limites e mudanças de escopo; ainda não identificado formalmente. |

## Responsabilidade resumida

O Sabiá recebe interações de usuários autorizados pelo Telegram e entradas locais explicitamente permitidas, registra contatos conhecidos, mantém um catálogo persistente de ações, aplica autorização e roteamento, aciona operações cadastradas, coordena jobs e verificações agendadas, mantém estado operacional e entrega mensagens ou mídias ao destino correspondente.

## Limites principais

- A responsabilidade remota começa quando um update é recebido por um cliente Telegram habilitado.
- A responsabilidade local começa quando uma aplicação autorizada utiliza uma interface publicada pelo Sabiá ou quando uma verificação cadastrada é disparada pelo scheduler.
- Operações originadas por usuários são limitadas a identificadores previamente cadastrados.
- Caminhos de scripts e executáveis são definidos apenas pelo operador local.
- A hierarquia do menu é derivada do caminho lógico persistido da ação.
- Sistemas e scripts acionados permanecem responsáveis pela própria lógica de negócio.

## Critérios de sucesso

- Um usuário autorizado consegue executar uma operação cadastrada e receber o resultado pelo cliente Telegram correto.
- Operações demoradas não bloqueiam o recebimento de novos comandos.
- Verificações agendadas conseguem gerar alertas por mudança de estado.
- O serviço não exige shell arbitrário, execução como root nem webhook público no primeiro MVP.
- Mídia pendente pode sobreviver a reinicializações.
- Clientes Telegram mantêm autorização e catálogo próprios.
- `/start` registra ou atualiza um contato e apresenta ações habilitadas.
- O operador consegue cadastrar uma ação e associá-la a um script pelo terminal.
- Uma ação assíncrona pode produzir progresso, resultado textual e arquivos correlacionados.

## Objetivos

- [OBJ-001 — Acionar operações locais cadastradas](objectives/OBJ-001-acionar-operacoes-locais-cadastradas.md)
- [OBJ-002 — Processar operações demoradas de forma assíncrona](objectives/OBJ-002-processamento-assincrono.md)
- [OBJ-003 — Executar verificações agendadas e alertas](objectives/OBJ-003-verificacoes-agendadas-e-alertas.md)
- [OBJ-004 — Ingerir e entregar mídia](objectives/OBJ-004-ingestao-e-entrega-de-midia.md)
- [OBJ-005 — Preservar estado operacional](objectives/OBJ-005-preservar-estado-operacional.md)
- [OBJ-006 — Restringir acesso e execução](objectives/OBJ-006-restringir-acesso-e-execucao.md)
- [OBJ-007 — Registrar contatos Telegram](objectives/OBJ-007-registrar-contatos-telegram.md)
- [OBJ-008 — Administrar catálogo de ações](objectives/OBJ-008-administrar-catalogo-de-acoes.md)

## Não objetivos

Ver [NON-OBJECTIVES.md](NON-OBJECTIVES.md).

## Restrições

Ver diretório [constraints/](constraints/).

## Premissas

Ver diretório [assumptions/](assumptions/).

## Questões abertas

- [OPEN-001 — Autoridade formal de aprovação](open-questions/OPEN-001-autoridade-formal-de-aprovacao.md)

## Scope

Ver [../scope/README.md](../scope/README.md).
