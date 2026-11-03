# Contexto do Sabiá

![DSG](https://img.shields.io/badge/DSG-DSG--0001-0550ae?style=flat-square)
![Status](https://img.shields.io/badge/Status-finalized-0a7ea4?style=flat-square)

## Objetivo

Representar o Sabiá no contexto do Telegram, dos usuários autorizados e do ambiente Linux/Bioma.

## Dependências

- [ADR-0002 — Múltiplos clientes Telegram isolados](../adr/telegram/multiplos-clientes-isolados.md)
- [ADR-0003 — Core independente do Telegram](../adr/arquitetura/core-independente-do-telegram.md)
- [ADR-0005 — Telegram Long Polling](../adr/telegram/long-polling.md)
- [ADR-0008 — Menor privilégio e autorização explícita](../adr/seguranca/menor-privilegio-e-autorizacao.md)

## Nível C4

`System Context`

## Diagrama

```text
Usuário autorizado
      |
      v
   Telegram
      |
      | Bot API / long polling
      v
+-----------------------------+
| Sabiá                       |
| - cliente Bioma             |
| - cliente Tools             |
| - cliente Alerts            |
| - Core                      |
+-----------------------------+
      |
      | operações previamente cadastradas
      v
Linux / serviços / aplicações Bioma
```

## Elementos e responsabilidades

- **Usuário autorizado:** inicia consultas e operações permitidas.
- **Telegram:** transporte externo das mensagens e arquivos.
- **Sabiá:** aplica autorização, roteia comandos, executa jobs, agenda verificações e produz alertas.
- **Linux/Bioma:** ambiente local no qual existem scripts, serviços e aplicações integradas.

## Relações relevantes

- o Telegram não recebe acesso a shell arbitrário;
- clientes Telegram mantêm autorização e catálogo próprios;
- o Sabiá inicia a comunicação necessária com a Bot API no modo long polling;
- integrações locais são acessadas por executores e interfaces do Core.

## Critérios para finalização

- fronteiras externas estão identificadas;
- não existe dependência do Core em bot específico;
- a ausência de porta pública para webhook está representada.
