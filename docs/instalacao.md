# Instalação — situação e requisitos

> **Não há instalação reproduzível nesta branch.** Em 03/11/2026, `develop` não possui `/src`, `go.mod`, executável, unit systemd ou migrações executáveis. Não executar comandos de compilação ou instalação como se estivessem validados.

## Ambiente previsto

| Componente | Definição documental | Situação |
|---|---|---|
| Sistema operacional | Debian 13 (Trixie), `amd64` | Perfil inicial; não validado em servidor |
| Serviço | Binário Go `sabia` supervisionado por systemd | Proposto |
| Identidade | Conta Linux `sabia`, sem root | Contrato documental |
| Banco | PostgreSQL com banco lógico e role próprios | Modelo e conexão ainda em refinamento |
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

## Referências técnicas

- [Modelo de configuração — CFG-0001](especificacao/configuracao/modelo-configuracao.md)
- [Serviço planejado — SVC-001](implementation/services/SVC-001-sabia/SVC-001-sabia.md)
- [Viabilidade Debian/systemd — DSC-003](charter/discovery/debian-systemd/DSC-003-viabilidade-do-servico-linux.md)
- [Pendências que impedem a implantação](pendencias.md)
- [Segurança e permissões planejadas](seguranca-acessos.md)
