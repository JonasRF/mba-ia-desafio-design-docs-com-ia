# ADR-003: Retry com backoff exponencial e Dead Letter Queue em tabela separada

## Status

Aceito

## Data

Decisão tomada em reunião técnica de kickoff da feature (ver `TRANSCRICAO.md`). ADR registrado em 2026-10-05.

## Decisores

Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)

## Contexto

Como os webhooks são entregues a sistemas de terceiros (`[09:02] Sofia`: "é outbound webhook"), é esperado que o endpoint do cliente esteja, eventualmente, indisponível, lento ou retornando erro. `[09:14]-[09:15] Larissa/Diego` discutiram a política de nova tentativa.

`[09:15] Diego` propôs backoff exponencial com um teto de tentativas, após o qual o evento é considerado falha permanente. `[09:15] Bruno` sugeriu 3 tentativas por ser "mais agressivo"; `[09:16] Diego` rejeitou essa proposta citando um caso real: um cliente já teve indisponibilidade de duas horas durante uma manutenção planejada, e 3 tentativas em 30 minutos matariam o evento antes da recuperação do cliente. O número de tentativas fechado foi **5**, com progressão **1 minuto, 5 minutos, 30 minutos, 2 horas, 12 horas** (`[09:17] Diego`), totalizando cerca de 15 horas entre a primeira falha e a última tentativa. `[09:17] Marcos` validou do ponto de vista de produto: um cliente fora do ar por 15 horas já tem um problema sério do lado dele.

Na sequência (`[09:17]-[09:18] Larissa/Diego`), foi decidido onde registrar eventos que esgotaram as tentativas: uma tabela separada, `webhook_dead_letter`, contendo payload, motivo da falha e timestamp — ao invés de apenas marcar o registro como "failed" na própria `webhook_outbox`. O argumento de Diego foi manter a outbox principal limpa (já que ela precisa ser varrida continuamente pelo worker, ver [ADR-002](./ADR-002-worker-dedicado-em-polling.md)) e ter uma tabela dedicada como evidência para debug e reprocessamento. `[09:18]-[09:19] Diego/Larissa` decidiram também que o reprocessamento de itens da DLQ é manual, via endpoint administrativo (`POST /admin/webhooks/dead-letter/:id/replay`), que recoloca o evento na outbox como pendente.

## Decisão

Implementar retry com **backoff exponencial** e teto de **5 tentativas**, com intervalos de **1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas** entre tentativas sucessivas. Após a 5ª tentativa falhar, o evento é movido para uma tabela dedicada `webhook_dead_letter` (payload, motivo da falha, timestamp), removido do fluxo ativo da `webhook_outbox`.

A reversão de um evento da DLQ para processamento é uma ação manual e explícita, via endpoint administrativo de replay, e não um mecanismo automático.

## Alternativas Consideradas

### Retry indefinido com backoff crescente, sem teto

Continuar tentando reenviar o evento indefinidamente, apenas aumentando o intervalo entre tentativas.

- Descartada porque mantém eventos "pendurados" para sempre caso o cliente tenha efetivamente desaparecido ou descontinuado o endpoint, sem um ponto de decisão claro de falha permanente (`[09:15] Diego`).

### Teto de 3 tentativas

Reduzir o número de tentativas para ser mais agressivo em declarar falha.

- Descartada porque uma janela curta (3 tentativas em ~30 minutos) falha precocemente diante de indisponibilidades legítimas e já observadas do lado do cliente, como manutenções planejadas de poucas horas (`[09:16] Diego`).

### Marcar falha permanente diretamente na tabela `webhook_outbox` (sem tabela DLQ separada)

Usar um status `failed` na própria outbox ao invés de mover o evento para uma tabela dedicada.

- Descartada porque mistura o fluxo ativo (que o worker varre continuamente) com o histórico de falhas permanentes, dificultando tanto a performance de leitura da outbox quanto a investigação e reprocessamento dedicados a falhas (`[09:18] Diego`).

## Consequências

### Positivas

- Tolerância a indisponibilidades reais e prolongadas do lado do cliente (até ~15 horas), reduzindo falsos positivos de "cliente morto".
- Separação clara entre fluxo operacional ativo (`webhook_outbox`) e histórico de falhas permanentes (`webhook_dead_letter`), cada um com seu próprio padrão de acesso e volume.
- Caminho de recuperação explícito e auditável via endpoint administrativo de replay, ao invés de reprocessamento automático potencialmente repetitivo.

### Negativas

- Um evento com falha persistente pode levar até ~15 horas para ser definitivamente classificado como falha, atraso que pode ser inaceitável para o cliente dependendo da criticidade do evento.
- A movimentação para a DLQ exige lógica adicional de leitura e escrita entre duas tabelas (outbox e dead letter), e mais um fluxo (replay) a manter e testar.
- Reprocessamento manual implica que, sem intervenção humana, eventos na DLQ nunca são reentregues automaticamente — depende de operação ativa de monitoramento.

### Trade-off explícito

Trocamos velocidade de declarar falha permanente por resiliência a indisponibilidades reais e prolongadas do cliente, aceitando que um evento problemático só é definitivamente isolado (e ainda assim reversível apenas manualmente) após quase um dia.
