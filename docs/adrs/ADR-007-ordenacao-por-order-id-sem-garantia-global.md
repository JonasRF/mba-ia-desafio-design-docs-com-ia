# ADR-007: Ordenação de entrega garantida apenas por order_id, dependente de single-worker

## Status

Aceito (limitação conhecida e documentada)

## Data

Decisão tomada em reunião técnica de kickoff da feature (ver `TRANSCRICAO.md`). ADR registrado em 2026-10-05.

## Decisores

Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos), Marcos (Product Manager)

## Contexto

`[09:12] Larissa` levantou a questão de ordenação: se um pedido muda de `PAID` para `PROCESSING` para `SHIPPED` em sequência rápida, o cliente recebe os eventos na ordem correta?

`[09:12] Diego` explicou que, com um único worker processando a `webhook_outbox` em ordem de `created_at` (ver [ADR-002](./ADR-002-worker-dedicado-em-polling.md)), a ordem de entrega é preservada. Porém, essa garantia depende inteiramente de haver **um único worker** ativo: se no futuro o sistema escalar para múltiplos workers processando a outbox em paralelo, a ordem de entrega por pedido deixa de ser garantida, a menos que se particione o processamento por `order_id` ou se use lock pessimista (`[09:13] Diego`) — explicitamente descrito como "problema do futuro, não agora".

`[09:13] Larissa` decidiu documentar isso como **limitação conhecida**: não há garantia de ordenação global entre pedidos diferentes, apenas ordenação por `order_id`, e apenas enquanto a arquitetura permanecer single-worker. `[09:14] Marcos` confirmou, do lado de produto, que os clientes B2B nunca pediram garantia de ordenação global — eles só precisam saber que cada pedido individual deles mudou de status.

## Decisão

Documentar e assumir como restrição de design que a garantia de ordenação de entrega de eventos de webhook é válida **apenas por `order_id`** (eventos do mesmo pedido chegam na ordem em que as mudanças de status ocorreram) e **apenas enquanto a arquitetura de processamento for single-worker**. Não há garantia de ordenação entre eventos de pedidos diferentes, nem garantia de ordenação de qualquer tipo caso a arquitetura evolua para múltiplos workers processando a outbox concorrentemente sem particionamento por `order_id` ou lock pessimista.

Esta decisão não introduz nenhum mecanismo técnico novo — é a aceitação explícita de uma limitação decorrente da decisão de execução single-worker (ADR-002), registrada para não ser perdida ou mal assumida como "ordenação garantida" por engenheiros futuros.

## Alternativas Consideradas

### Garantir ordenação global desde já (particionamento ou lock pessimista)

Implementar, já nesta fase, particionamento do processamento por `order_id` ou lock pessimista para permitir múltiplos workers em paralelo sem perder ordenação.

- Descartada para o escopo atual: não há requisito de negócio que demande ordenação global entre pedidos distintos (`[09:14] Marcos`), e a arquitetura single-worker já atende ao requisito real dos clientes (saber que o próprio pedido mudou de status, na ordem correta). Adicionar essa complexidade agora seria resolver um problema inexistente no presente (`[09:13] Diego`: "isso é problema do futuro, não agora").

## Consequências

### Positivas

- Mantém a implementação inicial simples (um único worker, uma fila ordenada por `created_at`), sem lógica adicional de particionamento ou locks.
- Atende integralmente ao requisito real dos clientes B2B, que é saber a sequência de mudanças de status de cada pedido individual, não uma ordenação cronológica entre pedidos de clientes diferentes.
- A limitação é documentada de forma explícita, reduzindo o risco de suposições incorretas por parte de times futuros que queiram escalar o worker sem revisitar esta decisão.

### Negativas

- Impõe um teto de escala: o sistema não pode evoluir para múltiplos workers em paralelo sem um redesenho dedicado (particionamento por `order_id` ou lock pessimista), que não foi especificado nesta fase.
- Se a necessidade de escalar aparecer sob pressão (ex.: outbox acumulando volume muito acima do esperado), a ausência de um plano de particionamento pronto pode forçar uma decisão apressada no futuro.

### Trade-off explícito

Aceitamos um teto de escalabilidade horizontal do worker (permanecer single-worker) em troca de simplicidade imediata de implementação, por não existir, hoje, requisito de negócio que justifique o custo de resolver ordenação sob concorrência.
