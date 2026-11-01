# Fronteira do sistema

### Onde começa

A responsabilidade do Sabiá começa quando ocorre um dos seguintes eventos:

- um cliente Telegram habilitado recebe um update;
- o operador invoca a CLI administrativa do próprio executável `sabia` para cadastrar ou administrar uma ação;
- uma aplicação local autorizada solicita uma operação por interface publicada pelo Sabiá;
- uma aplicação local autorizada inicia a ingestão de mídia;
- o scheduler interno identifica uma verificação cadastrada que deve ser executada;
- o serviço inicia ou reinicia e precisa recuperar trabalho operacional persistido.

### Onde termina

A responsabilidade do Sabiá termina, conforme o fluxo, quando:

- uma ação administrativa é validada e persistida no catálogo ou recusada de forma controlada;
- uma resposta ou mídia é aceita pelo Telegram e o resultado da entrega é registrado;
- uma requisição ou job chega a um estado terminal e o resultado correspondente é persistido;
- uma ingestão local é recusada de forma controlada ou aceita e persistida;
- uma verificação agendada atualiza o estado operacional e, quando aplicável, produz a notificação correspondente;
- o Sabiá registra falhas controladas sem transferir para o usuário acesso direto aos recursos internos.

A lógica específica executada por scripts, aplicações ou processadores externos não passa a fazer parte do domínio do Sabiá apenas por ser acionada por ele.
