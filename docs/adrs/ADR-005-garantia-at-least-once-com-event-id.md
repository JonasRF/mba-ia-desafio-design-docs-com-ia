# ADR-005: Garantia de entrega at-least-once com deduplicação via X-Event-Id

## Status

Aceito

## Data

Decisão tomada em reunião técnica de kickoff da feature (ver `TRANSCRICAO.md`). ADR registrado em 2026-10-05.

## Decisores

Diego (Eng. Sênior, Plataforma), Larissa (Tech Lead), Sofia (Eng. de Segurança), Bruno (Eng. Pleno, Pedidos)

## Contexto

Dado o modelo de outbox + worker com retry (ver [ADR-001](./ADR-001-outbox-no-mysql.md), [ADR-002](./ADR-002-worker-dedicado-em-polling.md) e [ADR-003](./ADR-003-retry-com-backoff-e-dead-letter-queue.md)), existem cenários em que o mesmo evento pode ser entregue mais de uma vez ao cliente — por exemplo, se o worker processa o envio, o cliente recebe e processa a requisição, mas a confirmação (resposta HTTP) se perde antes do worker marcar o evento como entregue, resultando em uma nova tentativa.

`[09:24] Diego` declarou explicitamente que a garantia de entrega será **at-least-once**: o cliente pode, eventualmente, receber o mesmo evento duas vezes, e precisa estar preparado para isso. `[09:25] Bruno` perguntou como o cliente diferencia duplicatas; `[09:25] Diego` definiu que cada evento carrega um **UUID único gerado no momento em que entra na outbox**, enviado no header `X-Event-Id`, que o cliente usa para deduplicar do lado dele.

`[09:25] Sofia` observou que essa abordagem "joga a responsabilidade para o cliente". `[09:25] Diego` reconheceu o trade-off, mas justificou a escolha citando precedente de mercado (Stripe, GitHub) e o custo desproporcional de implementar exactly-once, que exigiria coordenação de duas fases entre plataforma e cliente. `[09:26] Marcos` confirmou que documentará essa exigência de forma destacada no portal do desenvolvedor para os clientes.

## Decisão

Garantir semântica de entrega **at-least-once**: um evento pode, em cenários de falha de rede ou timeout na confirmação, ser entregue mais de uma vez ao mesmo endpoint de cliente. Cada evento recebe um identificador único (UUID) gerado no momento da inserção na `webhook_outbox`, propagado em todas as tentativas de entrega (inclusive retries) no header HTTP `X-Event-Id`. A responsabilidade de deduplicação do lado do recebedor (idempotência por `event_id`) é do cliente, e deve ser comunicada de forma explícita na documentação de integração (portal do desenvolvedor).

## Alternativas Consideradas

### Garantia exactly-once

Implementar um protocolo de confirmação em duas fases (ou equivalente) entre plataforma e cliente, de forma que cada evento seja processado exatamente uma vez, sem possibilidade de duplicata nem de perda.

- Descartada por exigir coordenação ativa de ambos os lados (plataforma e cliente) — um protocolo muito mais complexo de implementar e operar — para resolver um problema que o padrão de mercado (at-least-once + dedup por ID) já resolve para a vasta maioria dos casos práticos (`[09:25] Diego`: "Garantir exactly-once exigiria coordenação dos dois lados e fica muito mais complexo").

## Consequências

### Positivas

- Modelo simples de implementar e operar do lado da plataforma: não exige rastrear confirmações de processamento do cliente, apenas confirmação de entrega HTTP.
- Alinhado a um padrão de mercado já conhecido e documentado por provedores de referência (Stripe, GitHub), reduzindo a curva de aprendizado para os clientes B2B que já integram com outros webhooks.
- Combinado ao retry com backoff (ADR-003), favorece a confiabilidade de entrega (o evento quase certamente chega) em vez de arriscar perda de eventos em nome de evitar duplicatas.

### Negativas

- Transfere para o cliente a responsabilidade de implementar deduplicação por `event_id`, uma exigência de integração que pode ser mal interpretada, ignorada ou implementada incorretamente do lado dele.
- Se o cliente não deduplicar corretamente, mudanças de status podem ser processadas mais de uma vez no sistema dele (por exemplo, disparar duas vezes uma automação local), com consequências fora do controle da plataforma.

### Trade-off explícito

Aceitamos a possibilidade de entregas duplicadas — e a dependência de que o cliente implemente deduplicação corretamente — em troca de evitar a complexidade de coordenação exigida por exactly-once, seguindo um padrão de mercado já validado por outras plataformas de webhook.
