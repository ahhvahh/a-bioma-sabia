# DSC-004 — Viabilidade das interfaces locais

![Tipo: DSC](../../assets/documentation-style/badges/badge-type-dsc.svg) ![Status: refinement](../../assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](../../assets/documentation-style/badges/badge-version-v1.svg)

## Questão e contexto

A integração com este serviço/ambiente é necessária ao escopo previsto.

## Evidência documental conhecida

Aplicações locais devem operar com identidades Linux autorizadas, conforme [CON-007](../../conditions/CON-007-interfaces-locais-compartilhadas-com-aplicacoes-sao-limitada.md) e [ASM-005](../../conditions/ASM-005-aplicacoes-locais-que-utilizem-interfaces-compartilhadas-ser.md). A viabilidade de integrações efetivamente instaladas permanece sem comprovação.

## Perfil inicial recomendado para a primeira versão

- Manter **acesso negado por padrão** para identidades locais não autorizadas.
- Permitir explicitamente apenas usuários/grupos Linux necessários às interfaces locais. O grupo concreto, o mecanismo de associação e os contratos pertencem ao Scope/Implementation.
- Preservar isolamento entre o serviço Sabiá e clientes, sem expor banco, diretórios internos, tokens Telegram ou shell arbitrário aos consumidores.
- Caso sejam empregados **sockets Unix com caminho no sistema de arquivos**, controlar permissões do socket e dos diretórios pais e verificar efetivamente a conexão com usuário autorizado/não autorizado. Esta hipótese não fixa o transporte obrigatório nem autoriza sockets abstratos como substituto de controle de acesso.
- Permitir ajustes administrativos futuros de grupo e permissões, sem enfraquecer os limites de autorização do Charter.

## Evidências oficiais (consulta: 01/11/2026)

- Debian, `unix(7)` — permissões de sockets Unix e diretórios: https://manpages.debian.org/trixie/manpages/unix.7.en.html
- systemd.exec — `User=`, `Group=` e `SupplementaryGroups=`: https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html

## Método de validação da integração

1. Identificar a interface real e as identidades Linux escolhidas na etapa técnica.
2. Verificar acesso de cliente explicitamente autorizado e rejeição de usuário não autorizado.
3. Confirmar que acesso ao canal não concede acesso direto a diretórios internos ou dados do banco.
4. Verificar comportamento após reinício do serviço e alteração/revogação de acesso, quando aplicável.
5. Registrar as evidências de instalação e os resultados, sem executar alterações nesta etapa Charter.

**Nenhuma interface local real foi ensaiada nesta rodada.**

## Lacunas de verificação

- Identificar clientes reais, interface e permissões efetivas da instalação.
- Provar testes positivos/negativos de autorização e registrar condições de acesso antes de concluir o Discovery.
- Manter a definição de contratos e mecanismos de conexão no Scope/Implementation.

## Conclusão

**Mecanismos de isolamento conhecidos; integração ambiental inconclusiva — `refinement`.** As permissões Unix são compatíveis com o objetivo, mas falta uma integração real testada.

## Dependências

[CON-007](../../conditions/CON-007-interfaces-locais-compartilhadas-com-aplicacoes-sao-limitada.md), [ASM-005](../../conditions/ASM-005-aplicacoes-locais-que-utilizem-interfaces-compartilhadas-ser.md), [OBJ-004](../../objectives/OBJ-004-ingestao-e-entrega-de-midia.md); contratos técnicos e testes concretos seguem ao Scope/Implementation.
