<!-- A-BIOMA-DOC-STANDARD -->
<!-- A-BIOMA-DOC-REVISADO -->

# CONTR-011 — Requisitos da API administrativa UDS

**Situação:** `refinement` — requisitos recuperados do consolidado histórico, **sem contrato de fio aprovado e sem implementação verificada**. O marcador REVISADO atesta apenas a fidelidade da migração documental.

## Objetivo e limite

A API UDS administrativa é a interface local planejada para o `sabiactl` e outras aplicações autorizadas. O serviço `sabia` é a única camada que manipula PostgreSQL e diretórios internos. O operador **não acessa SQL diretamente**, mesmo podendo conectar ao socket.

**A definir antes de implementar:** caminho UDS, framing, serialização, versões do protocolo, nomes de operações, estruturas de requisição/resposta, paginação, códigos de erro, política de auditoria e autenticação/autorização por operação. **MessagePack está documentado para ingestão de mídia, não para esta API.** Este documento não define essas estruturas por inferência.

## Capacidades obrigatórias em nível funcional

| Área | Dados conceituais necessários | Resposta funcional esperada | Validações/falhas ainda a especificar |
|---|---|---|---|
| Saúde e conexões | Componente, tipo de sessão ou filtro | Estado, uptime, diagnósticos e contagens separadas de UDS, sessões locais, contatos Telegram e clientes configurados | Serviço indisponível, filtro inválido, falha de coleta |
| Serviços e processadores | Identificação e filtros autorizados | Estado e configurações conhecidas sem segredos | Não encontrado, autorização |
| Testes | Alvo e verificação permitida | Resultado, duração, erros | Permissão, timeout, falha de destino |
| Bots, usuários e solicitações | Bot, filtro e pedido selecionado | Bots e pendências por cliente; decisão de aprovação ou rejeição auditável | Paginação, reenvio, concorrência entre operadores |
| Menus e ações | Bot, identificação de ação, posição, script registrado e permissões | Cadastro ou publicação autorizada | Dados inválidos, conflito, script não validado |
| Configurações | Chave/valor permitido | Resultado da consulta/mutação sem expor segredos | Fonte de verdade YAML/SQL, autorização, entrada inválida |
| Envio de scripts | Arquivo local e metadados básicos | Registro de upload e resultado de validação | Tamanho, integridade, armazenamento, isolamento, erro de sintaxe |
| Incorporação e ativação | Identificador previamente validado | Estado da incorporação/ativação | Não validado, falta de permissão, confirmação humana ausente |

Nenhuma linha desta tabela estabelece **nome de método**, campos de payload ou sucesso operacional implementado.

## Controle de acesso e segurança

- O usuário do serviço previsto é `sabia`, sem execução como `root`. O grupo `absabia` permite conexão aos sockets de aplicação **quando aprovado**, mas não concede privilégio administrativo automático.
- Operações privilegiadas exigem identidade local, autorização específica, revalidação no serviço e auditoria.
- Os sockets de mídia possuem conflito histórico de grupo `abioma` versus `absabia`. O acesso ao PostgreSQL usa grupo proposto `abioma_db` e role SQL própria; não herdar esse acesso para o `sabiactl`.
- Não revelar credenciais, tokens ou conteúdo de arquivos em logs ou respostas.
- Decisões do operador (aprovação e rejeição) devem exigir confirmação `Y`; `N` ou `Esc` cancelam sem mudar o estado.

## Pendências de consistência

- Resolver precedência entre `telegram.clients.allowed_users` no YAML com reinício e autorizações persistentes aprovadas dinamicamente pelo terminal.
- Aprovar o schema de contatos, solicitações, scripts e ações antes de inferir operações SQL. As entidades atuais são propostas.
- Definir concorrência e proteção contra decisão dupla por operadores diferentes; exigir registro transacional da decisão.
- Definir limites e isolamento da validação dos scripts. `bash -n` valida sintaxe de Bash, **não comprova segurança**.

## Vínculos

- [Inventário de canais](../arquitetura/ARQ-011-inventario-canais-servicos.md)
- [Terminal administrativo](../interfaces/terminal/INTF-TERM-001-sabiactl.md)
- [Fluxo de aprovação por bot](../fluxos/FLUX-011-aprovacao-acesso-telegram.md)
- [Fluxo de validação/publicação de script](../fluxos/FLUX-012-validacao-publicacao-script.md)
- [Segurança e autorização](../arquitetura/ARQ-006-seguranca-autorizacao.md)
- [Gate de especificação](../referencia/REF-006-gate-mvp.md)

## Rastreabilidade

Fonte: [Regras consolidadas — seções 2, 3, 4 e 11](../regras-consolidadas-sabia.md), [inventário histórico de interfaces](../interfaces-servicos.md). O contrato permanece em `refinement`; **nenhuma escolha técnica pendente foi transformada em aprovação**.
