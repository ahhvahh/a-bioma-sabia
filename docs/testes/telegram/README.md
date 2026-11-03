# Plano de testes locais — Telegram/Sabiá

**Data:** 2026-10-21  
**Estado:** casos de teste especificados, aguardando implementação do adaptador em `/src`.

## Organização

Criar futuramente testes automatizados com estes grupos (identificadores estáveis). Rodar cada grupo separadamente, sem token real e sem depender da rede Telegram. Utilizar um servidor HTTP fake da Bot API e PostgreSQL de teste com reset por caso.

| Grupo | Arquivo de cenários | Objetivo |
|---|---|---|
| T01 | [T01-inbox.md](T01-inbox.md) | Atomicidade, crash recovery, idempotência |
| T02 | [T02-menu.md](T02-menu.md) | Menus, callbacks, autorização |
| T03 | [T03-midia.md](T03-midia.md) | Limites, MIME e métodos de envio |
| T04 | [T04-retry.md](T04-retry.md) | 429, 5xx, falha ambígua, restart |
| T05 | [T05-isolamento.md](T05-isolamento.md) | Três clientes isolados |

## Processo de execução local esperado

1. Iniciar banco PostgreSQL descartável e servidor Bot API fake.
2. Criar schema e carregar fixture JSON de cada caso.
3. Rodar isoladamente o grupo T01, T02, T03, T04 ou T05.
4. Registrar `PASS/FAIL`, resultado persistido e chamadas capturadas pelo fake.
5. Repetir com falhas injetadas nos pontos de transação, transporte e restart.

**Não há comando Go de testes executável no Git hoje:** a branch `develop` não contém `/src`. Ao implementar, fornecer `go test ./...` e flags/tags para executar individualmente os grupos, além de scripts em pasta de testes do código. Não colocar código de testes executáveis dentro de `/docs`; esta pasta contém contratos e casos de teste.
