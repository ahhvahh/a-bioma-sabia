<!-- A-BIOMA-DOC-LEGADO -->
<!-- A-BIOMA-DOC-MIGRADO -->

# Instalação — situação e requisitos

> **Conteúdo lido e migrado para:** [GUIA-001 — documento canônico](guias/GUIA-001-preparar-instalacao.md). O texto histórico permanece disponível para rastreabilidade; especificações em aberto não foram aprovadas.

> **Não há instalação reproduzível nesta branch.** Em 03/11/2026, `develop` não possui `/src`, `go.mod`, executável, unit systemd ou migrações executáveis. Não executar comandos de compilação ou instalação como se estivessem validados.

## Ambiente previsto

| Componente | Definição documental | Situação |
|---|---|---|
| Sistema operacional | Debian 13 (Trixie), `amd64` | Perfil inicial; não validado em servidor |
| Serviço | Binário Go `sabia` supervisionado por systemd | Proposto |
| Identidade | Conta Linux `sabia`, sem root; grupos `absabia` e `abioma_db` | Contrato documental |
| Banco | PostgreSQL com banco lógico e role próprios; conexão via UDS restrito a `abioma_db` | Modelo e conexão ainda em refinamento |
| Telegram | Tokens de clientes lógicos, referências por variáveis de ambiente | Previsto |
| Arquivo operacional | `/etc/sabia/sabia.yaml` | Estrutura em refinamento |
| Ingestão de mídia | Dois Unix sockets distintos | Caminhos definitivos pendentes |

## Etapas para uma implantação futura

1. Concluir os contratos e pendências documentais necessários ao MVP.
2. Implementar o serviço, testes, migrações, CLI e empacotamento.
3. Validar os requisitos no ambiente Debian e PostgreSQL alvo.
4. Provisionar conta dedicada, permissões, diretórios e credenciais sem expor segredos.
5. Compilar, instalar a unit systemd e verificar inicialização, parada, recuperação e isolamento.

**Essas etapas são um roteiro de preparação, não comandos já disponíveis.** O procedimento operacional completo com comandos reais deverá ser produzido após a implementação e a validação; por isso não existe ainda um `instalacao-configuracao.md` executável.

## Isolamento do serviço e preparação UDS — requisito novo

- Criar futuramente o usuário de serviço `sabia` sem login interativo e sem root.
- Criar os grupos `absabia` (clientes UDS) e `abioma_db` (UDS PostgreSQL); associar `sabia` a ambos, e os operadores comuns somente a `absabia`.
- Configurar os sockets da aplicação com owner `sabia`, grupo `absabia`, modo planejado `0660` e diretórios protegidos.
- Configurar PostgreSQL com socket no diretório restrito por `abioma_db`, autenticação local adequada em `pg_hba.conf` e role SQL exclusiva; impedir acesso não autorizado por TCP.
- Instalar futuramente `sabiactl` como cliente de terminal das APIs UDS; ele nunca acessará o banco ou os arquivos internos diretamente.
- Validar que operador `absabia` consegue acessar a API, mas não o banco; usuário sem `absabia` é bloqueado; o processo `sabia` consegue conectar ao PostgreSQL via UDS.

Essas são etapas **planejadas**, sem comandos de instalação comprovados. Configuração de socket do PostgreSQL tem escopo de instância/cluster, exigindo revisão especial em ambientes compartilhados.

## Referências técnicas

- [Modelo de configuração — CFG-0001](especificacao/configuracao/modelo-configuracao.md)
- `SVC-001 — Serviço planejado` (documento histórico não recuperado nesta branch)
- `DSC-003 — Viabilidade Debian/systemd` (documento histórico não recuperado nesta branch)
- [Pendências que impedem a implantação](pendencias.md)
- [Segurança e permissões planejadas](seguranca-acessos.md)
