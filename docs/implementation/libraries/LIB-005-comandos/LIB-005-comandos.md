# LIB-005 — Biblioteca Comandos
![Tipo: LIB](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-type-lib.svg) ![Status: refinement](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](https://raw.githubusercontent.com/ahhvahh/doc-arquitetura/652e1035a6ebb7ade0f21a6f2b9d08156ba35b37/ProjectCharter/1.0/assets/documentation-style/badges/badge-version-v1.svg)

## Responsabilidade

Roteamento de operações previamente registradas; organização dos menus hierárquicos e botões de navegação.

## Contratos de origem

- [IN-011](../../../01-Project_Charter/scope/in-scope/IN-011-catalogo-persistente-de-acoes-e-menu.md) — *refinement*.
- [FLW-007](../../../01-Project_Charter/scope/flows/FLW-007-cadastro-administrativo-e-execucao-de-acao-persistente.md) — *refinement*.
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
    A["SVC-001 Sabiá"] --> B["LIB-005 Comandos"]
    B --> C["Contrato Scope PENDING"]
```

### Diagrama de sequência proposto (não normativo)

```mermaid
sequenceDiagram
    participant S as SVC-001
    participant L as Comandos
    S->>L: solicitação, contrato PENDING
    L-->>S: resposta, TYPE PENDING
```

## Error Handling

Ação inválida, expirada ou desconhecida não autoriza execução. Representações, códigos e retentativas permanecem `PENDING` até contratos de erro refinados.

## Observability

- Trace: detalhe técnico opcional, sem payload sensível.
- Information: eventos operacionais relevantes.
- Warning: rejeições e falhas recuperáveis.
- Error: falha que impede uma operação solicitada.
- Critical: indisponibilidade da responsabilidade essencial.

Não registrar tokens, segredos nem anexos brutos.

## Validação e dependências

Casos correlatos do [plano T01–T05](../../../testes/telegram/README.md): T02, T05. Catálogo próprio `LIB-005-Txxx`, mocks e dependências concretas `DEP-*` ainda **PENDING**. Nenhum teste foi executado.

**Gate para Development:** BLOCKED até Scope e especificação técnica serem refinados. Nenhum código `/src` foi gerado.
