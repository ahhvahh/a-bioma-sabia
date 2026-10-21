# Sabiá — Implementation
![Tipo: GUIDE](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-type-guide.svg) ![Status: refinement](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-version-v1.svg)

Estrutura documental **proposta**, aprovada para Debian Linux `amd64` com linguagem recomendada Go, binário único `sabia` e supervisão systemd.

- [Serviço](services/README.md)
- [Bibliotecas](libraries/README.md)
- [Charter legado](../01-Project_Charter/charter/README.md)
- [Scope legado](../01-Project_Charter/scope/README.md)
- [Pendências documentais](../pendencias.md)
- [Contrato operacional Telegram](../telegram-operacao-2026.md)
- [Testes de referência T01–T05](../testes/telegram/README.md)

## Unidades

```mermaid
flowchart TD
    SVC["SVC-001 Serviço Sabiá (Go/amd64)"] --> L1["LIB-001 Telegram"]
    SVC --> L2["LIB-002 Configuração"]
    SVC --> L3["LIB-003 Persistência"]
    SVC --> L4["LIB-004 Segurança"]
    SVC --> L5["LIB-005 Comandos"]
    SVC --> L6["LIB-006 Mídia"]
    L1 --- API["Telegram Bot API"]
    L3 --- PG["PostgreSQL"]
```

**BLOCKED — Gate Scope → Implementation:** os Charter/Scope usados por estas unidades seguem em `refinement`; portanto esta organização é **PROPOSED**, não `refined` e muito menos `implemented`. TYPEs, ALG, interfaces, modelos físicos, migrações e dependências externas ficam `PENDING` nos arquivos das unidades. Não criar código `/src` com base nesta estrutura sem refinar as origens.

**PENDING visual:** não existem badges SVG locais sob `/docs/assets` no Sabiá; os badges destas propostas usam o catálogo original imutável da norma, até sua instalação autorizada no caminho canônico.

**Norma consultada:** `ahhvahh/doc-arquitetura` `main` em `652e1035a6ebb7ade0f21a6f2b9d08156ba35b37`. Sem alteração na referência.
