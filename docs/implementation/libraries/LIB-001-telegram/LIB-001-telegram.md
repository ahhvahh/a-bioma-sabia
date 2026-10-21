# LIB-001 — Biblioteca Telegram
![Tipo: LIB](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-type-lib.svg) ![Status: refinement](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-version-v1.svg)

## Responsabilidade

Integração HTTPS com Telegram Bot API: long polling, callback_query, menus e envio de mensagens; não define permissões de negócio.

## Contratos de origem

- [INT-001] (../../../01-Project_Charter/scope/integrations/INT-001-telegram-bot-api.md) — *refinement*.
- [FLW-001] (../../../01-Project_Charter/scope/flows/FLW-001-comando-telegram.md) — *refinement*.
- [Regras técnicas do Sabiá](../../../telegram-operacao-2026.md) — orientação anterior; não substitui Scope refinado.

**BLOCKED:** os contratos citados não estão `refined` nem `implemented`. A unidade define uma fronteira de responsabilidade aprovada para organização, **não** contrato operacional aprovado.

## Linguagem recomendada e target

- Go, para integrar a unidade ao binário único do serviço sem runtime adicional.
- Linux `amd64` (Debian Intel/AMD 64 bits).
- Alternativas C/C++/Rust exigiriam integração FFI e build adicional; não escolhidas pelo usuário.
- As Libraries serão ligadas ao serviço, não distribuídas como processos separados.

## Contrato público e estratégia

**PENDING:** TYPEs concretos, API Go, entradas e saídas, ALGs funcionais e técnicos, dependências DEP e política final de erros dependem do refinamento de Scope. Não inventar assinaturas nem estados.

### Fluxograma proposto (composição)

```mermaid
flowchart LR
    A["SVC-001 Sabiá"] --> B["LIB-001 Telegram"]
    B --> C["Contrato Scope PENDING"]
```

### Diagrama de sequência proposto (não normativo)

```mermaid
sequenceDiagram
    participant S as SVC-001
    participant L as Telegram
    S->>L: solicitação, contrato PENDING
    L-->>S: resposta, TYPE PENDING
```

## Error Handling

Não registrar tokens; falha na leitura não confirma update ainda não persistido. Representações, códigos e retentativas permanecem `PENDING` até contratos de erro refinados.

## Observability

- Trace: detalhe técnico opcional, sem payload sensível.
- Information: eventos operacionais relevantes.
- Warning: rejeições e falhas recuperáveis.
- Error: falha que impede uma operação solicitada.
- Critical: indisponibilidade da responsabilidade essencial.

Não registrar tokens, segredos nem anexos brutos.

## Validação e dependências

Casos correlatos do [plano T01–T05](../../../testes/telegram/README.md): T01, T02, T04, T05. Catálogo próprio `LIB-001-Txxx`, mocks e dependências concretas `DEP-*` ainda **PENDING**. Nenhum teste foi executado.

**Gate para Development:** BLOCKED até Scope e especificação técnica serem refinados. Nenhum código `/src` foi gerado.
