# Segurança e acessos — desenho e limites

> **Planejamento, não configuração aplicada.** Esta branch não contém unidade systemd, contas provisionadas, scripts de instalação ou código de autorização.

## Identidades e fronteiras

| Identidade | Função planejada | Acesso |
|---|---|---|
| `sabia` (usuário Linux) | Processo principal do serviço, não root | Configuração protegida, recursos internos e conexão ao PostgreSQL |
| `abioma` (grupo Linux) | Consumidores locais autorizados | Unix sockets de mídia e link de invocação local autorizado |
| Usuário do Telegram | Consumidor remoto | Apenas ações autorizadas para o cliente, usuário e chat |
| Role PostgreSQL do Sabiá | Identidade do banco de dados | Privilégios mínimos sobre o banco próprio; nome e provisão pendentes |

**Membro de `abioma` não recebe por essa condição acesso ao PostgreSQL**, credenciais SQL, diretórios internos ou permissão de assumir a identidade `sabia`. A documentação atual **não estabelece um grupo Linux separado chamado `abioma_db`**; criá-lo aqui seria uma decisão nova e não comprovada. Essa separação pode ser avaliada na etapa de implantação.

## Regras previstas para os sockets

| Controle | Valor documental |
|---|---|
| Owner | `sabia` |
| Group | `abioma` |
| Mode | `0660` |
| Transporte | Unix domain socket local, sem listener TCP |
| Caminhos definitivos | **BLOCKED** em CFG-0001 |

O produtor precisa tanto de acesso ao socket quanto de um `request_id` válido. `request_id` isolado não é credencial de acesso.

## Telegram, configuração e segredos

- Clientes lógicos separados: `bioma`, `tools` e `alerts`.
- Autorizar por Telegram User ID e, quando configurado, chat; revalidar callbacks.
- Ações e scripts só podem ser resolvidos pelo cadastro local; nenhuma entrada Telegram pode virar shell arbitrário.
- Arquivo YAML previsto em `/etc/sabia/sabia.yaml`, com `token_env` apontando para segredos protegidos; valores reais não vão ao Git ou aos logs.
- O serviço deve operar sem root, com privilégios e acesso aos diretórios limitados.
- A especificação de grupo, conta, permissões de diretório, conexão SQL e unit systemd **ainda precisa de implementação e validação operacional**.

Sem código ou serviço instalado, não há comandos de criação de conta, associação a grupo, `systemctl` ou validação de permissões que possam ser apresentados como procedimento testado.

Referências: [módulo de segurança](especificacao/modulos/seguranca.md), [ADR de menor privilégio](adr/seguranca/menor-privilegio-e-autorizacao.md), [CFG-0001](especificacao/configuracao/modelo-configuracao.md), [serviço planejado](implementation/services/SVC-001-sabia/SVC-001-sabia.md).
