# ADR-008: Payload do evento renderizado (snapshot) no momento da inserção na outbox

## Status

Aceito

## Data

Decisão tomada em reunião técnica de kickoff da feature (ver `TRANSCRICAO.md`). ADR registrado em 2026-10-05.

## Decisores

Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)

## Contexto

Ao final da reunião, já com a maior parte da arquitetura fechada, `[09:51] Bruno` levantou uma dúvida de modelagem: o evento registrado na `webhook_outbox` deve guardar o **payload já renderizado** (os dados do pedido no momento da mudança de status) ou apenas o `order_id`, renderizando o payload completo somente no momento do envio pelo worker?

`[09:52] Larissa` decidiu por armazenar o payload **já renderizado no momento da inserção** (snapshot): se o pedido mudar novamente antes do evento ser efetivamente entregue (por exemplo, por causa de retries ao longo de várias horas, ver [ADR-003](./ADR-003-retry-com-backoff-e-dead-letter-queue.md)), o evento entregue ainda deve refletir o estado do pedido **no instante em que aquela mudança de status especificamente ocorreu**, e não o estado atual (possivelmente já diferente) do pedido no momento do envio. `[09:52] Diego` concordou, reforçando que renderizar sob demanda criaria "casos esquisitos" de inconsistência entre o evento (ex.: `from_status`/`to_status` de uma transição antiga) e o estado real do pedido no momento do envio.

## Decisão

O payload do evento de webhook é **renderizado e persistido como snapshot** no momento em que a linha é inserida na `webhook_outbox`, dentro da mesma transação de `OrderService.changeStatus` (ver [ADR-001](./ADR-001-outbox-no-mysql.md)). O worker, ao processar o evento, **não** busca o estado atual do pedido para montar o payload — ele apenas lê e envia o payload já armazenado na outbox, inalterado, independentemente de quanto tempo se passou ou de quantas vezes o pedido mudou de status desde então.

## Alternativas Consideradas

### Renderizar o payload sob demanda, no momento do envio

Armazenar na outbox apenas uma referência (`order_id`, `from_status`, `to_status`) e montar o payload completo (dados do pedido) consultando o estado atual do pedido no momento em que o worker efetivamente envia a requisição HTTP.

- Descartada porque, em um cenário de retry com backoff que pode se estender por até ~15 horas (ADR-003), o pedido pode ter sofrido novas mudanças de status entre o momento do evento original e o momento do envio efetivo. Renderizar sob demanda faria o evento entregue refletir um estado do pedido que não corresponde mais à transição que o evento deveria representar, gerando inconsistência percebida pelo cliente (`[09:52] Larissa`, `[09:52] Diego`).

## Consequências

### Positivas

- Cada evento entregue é fiel ao estado exato do pedido no instante da transição de status que o originou, independentemente de quanto tempo se passou até a entrega (relevante dado o backoff de até 12h, ADR-003).
- Simplifica o worker: ele não precisa consultar o estado atual do pedido nem lidar com a possibilidade de o pedido já ter sido alterado ou até removido — apenas envia o payload armazenado.
- Reduz carga de leitura no banco durante o envio: não há necessidade de um `JOIN` ou consulta adicional ao pedido no momento do processamento pelo worker.

### Negativas

- Aumenta o tamanho de cada linha da `webhook_outbox`, já que o payload completo (ainda que enxuto, sem `items`, conforme discutido em `[09:43]-[09:44] Diego`) é duplicado e armazenado por evento, em vez de apenas uma referência.
- Qualquer erro de renderização do payload no momento da inserção (ex.: um campo calculado incorretamente) fica congelado no evento e só pode ser corrigido com uma nova mudança de status ou reprocessamento manual — não se "autocorrige" ao reenviar.

### Trade-off explícito

Aceitamos maior consumo de armazenamento por evento (payload completo persistido em vez de apenas uma referência) em troca de consistência temporal: o cliente sempre recebe o estado do pedido correspondente exatamente ao momento da transição notificada, mesmo que a entrega seja postergada por horas devido a retries.
