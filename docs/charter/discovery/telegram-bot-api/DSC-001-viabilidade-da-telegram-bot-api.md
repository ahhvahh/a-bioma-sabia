# DSC-001 — Viabilidade do Telegram Bot API

![Tipo: DSC](../../assets/documentation-style/badges/badge-type-dsc.svg) ![Status: refinement](../../assets/documentation-style/badges/badge-status-refinement.svg) ![Versão: v1](../../assets/documentation-style/badges/badge-version-v1.svg)

## Questão e contexto

A integração com este serviço/ambiente é necessária ao escopo previsto.

## Evidência documental conhecida

O `getUpdates` viabiliza recepção por long polling e não opera enquanto houver webhook configurado. A API documenta `limit` de 1 a 100 (padrão 100) e `timeout` padrão 0 (short polling); long polling exige valor positivo. A lista `allowed_updates`, se omitida, reutiliza a configuração anterior. O `offset` confirma updates anteriores e a resposta de controle de fluxo pode informar `retry_after`.

## Fontes oficiais (consulta em 01/11/2026)

- Telegram Bot API, métodos `getUpdates`, `getWebhookInfo`, `deleteWebhook` e `ResponseParameters` — https://core.telegram.org/bots/api (Bot API 10.3, 24/08/2026).
- Telegram Bots FAQ, confirmação de updates e limites de mensagens — https://core.telegram.org/bots/faq.

## Perfil inicial recomendado para a primeira versão

O responsável pelo projeto decidiu iniciar com valores padrão ajustáveis e revisá-los conforme a necessidade. Os números abaixo são **proposta operacional de referência**, não limites obrigatórios do Charter nem contrato de configuração; a especificação de parâmetros, persistência e ajustes pertence ao Scope/Implementation.

| Parâmetro | Valor inicial recomendado | Fundamentação |
|---|---|---|
| `getUpdates.timeout` | 30 segundos | Escolha inicial para long polling; diferente do padrão 0 da API. |
| `getUpdates.limit` | 100 updates | Padrão oficial; faixa aceita 1–100. |
| `allowed_updates` | `message`, `callback_query` | Cobertura inicial dos comandos e botões; revisar se novos eventos forem necessários. |
| `offset` | Próximo ID após o último update aceito com segurança | Evitar confirmações prematuras e repetição indevida; estado e algoritmo serão definidos no Scope. |
| Webhook | Não utilizar na primeira versão | Preserva CON-002 e a exclusividade do long polling. |
| Retentativas e limites | Respeitar `retry_after` e os limites retornados pelo provedor | Não supor quotas universais nem mascarar erros de autenticação. |

A configuração futura poderá ajustar os parâmetros sem mudar o propósito, os objetivos ou as restrições do Charter. Não é permitido omitir autorização, expor tokens em logs ou descartar updates pendentes automaticamente apenas por se adotarem padrões.

## Método de validação e critérios de aceite

1. Em ambiente de teste com token fornecido por meio seguro, confirmar acesso HTTPS de saída a `api.telegram.org`, executar `getMe` e consultar `getWebhookInfo`.
2. Verificar que não há webhook ativo antes de usar `getUpdates`; não remover webhook existente nem descartar updates sem decisão administrativa explícita.
3. Confirmar recepção de `/start` e `callback_query`, tratamento de updates repetidos e avanço seguro de `offset` com um bot de teste.
4. Simular indisponibilidade de rede, credencial inválida e limite de requisições; verificar recuperação e respeito a `retry_after`, sem revelar o token.
5. Registrar a evidência, data, ambiente, resultado e configuração efetiva antes de declarar viabilidade de implantação comprovada.

Nenhum ensaio de rede ou uso de credenciais foi executado nesta rodada.

## Lacunas de verificação

- Executar os ensaios acima em uma instalação real ou homologada; confirmar credenciais, conectividade e compatibilidade com a operação prevista.
- Validar no Scope/Implementation a especificação configurável, a confirmação segura de updates e a integração com múltiplos clientes.
- Não considerar viabilidade comprovada sem ensaio ou fonte oficial conclusiva aplicada ao ambiente.

## Conclusão

**Viável em documentação; implantação ainda inconclusiva — `refinement`.** A API pública suporta o perfil inicial proposto, mas faltam ensaios reproduzíveis no ambiente previsto. Não liberar o gate apenas com estes valores.

## Dependências

CON-002; ASM-001/ASM-002; OBJ-001/OBJ-007/OBJ-009; encaminhar consequências apenas aos documentos canônicos do Charter e ao Scope quando migrado.
