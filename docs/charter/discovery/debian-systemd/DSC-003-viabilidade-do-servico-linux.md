# DSC-003 — Viabilidade do serviço Linux

![Tipo: DSC](../../assets/documentation-style/badges/badge-type-dsc.svg) ![Status: refinement](../../assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](../../assets/documentation-style/badges/badge-version-v1.svg)

## Questão e contexto

A integração com este serviço/ambiente é necessária ao escopo previsto.

## Evidência documental conhecida

O [CON-001](../../conditions/CON-001-o-servico-opera-em-linux-e-o-executavel-principal-e-distribu.md) prevê serviço Linux gerenciado pelo systemd; o [CON-003](../../conditions/CON-003-o-processo-principal-nao-executa-como-root-e-utiliza-identid.md) exige identidade Linux dedicada sem execução como root. A escolha do usuário para a primeira versão é **Debian 13 (Trixie)** com ajustes posteriores conforme a necessidade.

## Perfil inicial recomendado para a primeira versão

| Aspecto | Padrão de referência | Observação |
|---|---|---|
| Distribuição | Debian 13 (Trixie) | Revisões de segurança e manutenção da própria versão continuam permitidas. |
| Gerência do serviço | systemd | Executar e reiniciar o processo do Sabiá sob uma unidade de sistema. |
| Identidade do processo | Usuário Linux de serviço dedicado, **não root** | Nome efetivo e política de provisionamento pertencem à Implementation. |
| Grupos | Grupo primário próprio e acesso adicional mínimo, quando necessário | Nenhum usuário externo recebe acesso por padrão. |
| Diretórios | Donos, grupos e permissões explícitos; sem escrita indiscriminada | Diretórios de estado e execução podem ser gerenciados pelo systemd. |
| Parâmetros | Ajustáveis conforme exigência comprovada do ambiente | A versão inicial não autoriza ampliar privilégios automaticamente. |

Estes padrões registram a intenção operacional, **não são um arquivo de unidade systemd nem especificação de instalação**. A Implementation definirá caminhos, usuário/grupo concretos, `User=`, `Group=`, diretórios e políticas de proteção; o Scope estabelecerá os contratos locais necessários.

## Evidências oficiais (consulta: 01/11/2026)

- Debian, identificação do Debian 13 como Trixie: https://wiki.debian.org/DebianTrixie
- Debian, pacote systemd no Trixie: https://packages.debian.org/trixie/systemd
- systemd, isolamento, identidade de processo e diretórios: https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html

## Método de validação da instalação

1. Confirmar versão real de Debian, presença do systemd e a identidade efetiva do processo.
2. Demonstrar que o serviço inicia, para e reinicia sem executar como root.
3. Verificar permissões de diretórios, ausência de acesso indevido a credenciais e condições de recuperação após reinício.
4. Registrar evidências reproduzíveis do ambiente (versões, configuração efetiva sem segredos e resultados).

**Nenhum ensaio de implantação no servidor foi executado nesta rodada.**

## Lacunas de verificação

- Testar os critérios acima no ambiente de implantação e preservar resultados verificáveis.
- Definir e validar no Scope/Implementation a configuração efetiva, sem transformar exemplos em contratos de Charter.
- Não alegar execução sem root ou viabilidade ambiental comprovada apenas com evidência documental.

## Conclusão

**Perfil inicial documentado; implantação inconclusiva — `refinement`.** Debian 13 e systemd suportam o modelo pretendido, mas falta validação do ambiente real.

## Dependências

[CON-001](../../conditions/CON-001-o-servico-opera-em-linux-e-o-executavel-principal-e-distribu.md), [CON-003](../../conditions/CON-003-o-processo-principal-nao-executa-como-root-e-utiliza-identid.md), [OBJ-006](../../objectives/OBJ-006-restringir-acesso-e-execucao.md); especificações de unidade, paths e permissões seguem ao Scope/Implementation.
