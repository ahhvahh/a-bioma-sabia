# T02 — Menu e callbacks

- T02-01: /start apresenta menu autorizado.
- T02-02: /menu apresenta as mesmas ações após atualização do catálogo.
- T02-03: categoria abre submenu e edita mensagem.
- T02-04: Voltar navega ao pai; Início à raiz.
- T02-05: cada clique chama answerCallbackQuery.
- T02-06: callback_data inválido, maior que 64 bytes ou adulterado → não executa.
- T02-07: permissão revogada após criação do menu → clique negado.
- T02-08: clique em mensagem inacessível → fallback para nova mensagem.
- T02-09: callback duplicado → operação não executada novamente.
