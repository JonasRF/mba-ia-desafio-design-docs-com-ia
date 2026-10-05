# ADR-001: Padrão Outbox no MySQL para publicação de eventos de webhook

## Status

Aceito

## Data

Decisão tomada em reunião técnica de kickoff da feature (ver `TRANSCRICAO.md`). ADR registrado em 2026-10-05.

## Decisores

Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)

## Contexto

A feature de notificação de pedidos precisa disparar um evento HTTP para sistemas externos (clientes B2B) sempre que o status de um pedido muda. O ponto de mudança de status é `OrderService.changeStatus` (`src/modules/orders/order.service.ts`), que já executa, dentro de uma única transação Prisma (`this.prisma.$transaction`), a atualização de `orders`, a criação de um registro em `order_status_history` e o débito/reposição de `stock_quantity` em `products`.

Em `[09:03] Larissa` foi levantada a pergunta central: disparar o webhook de forma síncrona dentro dessa transação, ou via algum mecanismo assíncrono (fila/outbox)?

`[09:04] Bruno` argumentou contra o disparo síncrono: a transação de mudança de status já é pesada, e adicionar uma chamada HTTP no meio dela significa que um cliente externo lento trava a mudança de status de outros pedidos. Além disso, não há como fazer rollback de uma notificação já enviada se o cliente estiver fora do ar.

`[09:06] Diego` propôs explicitamente o padrão Outbox: inserir uma linha em uma tabela `webhook_outbox` dentro da mesma transação SQL que já atualiza `orders` e `order_status_history`. Um worker separado lê essa tabela de forma assíncrona e dispara as chamadas HTTP. Se a transação principal commita, o evento foi registrado de forma garantida; se há rollback, o evento desaparece junto — eliminando o cenário de inconsistência entre estado do pedido e eventos publicados.

`[09:07] Diego` justificou o uso do MySQL existente (ao invés de subir nova infraestrutura) pelo porte da equipe: "a gente é um time pequeno. Subir Redis Cluster pra isso é overengineering."

## Decisão

Adotar o padrão **Transactional Outbox** usando uma tabela `webhook_outbox` no MySQL já utilizado pela aplicação (`prisma/schema.prisma`, datasource `mysql`). A inserção na outbox ocorre dentro da mesma transação Prisma que hoje executa a mudança de status em `OrderService.changeStatus`, via uma função `publishWebhookEvent(tx, order, fromStatus, toStatus)` que recebe o cliente de transação (`tx: Prisma.TransactionClient`) já em uso (`[09:41] Bruno`, `[09:41] Diego`).

A tabela deve ter índice em campo de status do evento (pendente, processando, falhou, entregue) e em `created_at`, para permitir leitura eficiente dos pendentes mais antigos pelo worker (`[09:07] Diego`).

## Alternativas Consideradas

### Disparo síncrono dentro da transação de `changeStatus`

Chamar o endpoint do cliente via HTTP diretamente dentro da transação que muda o status do pedido.

- Descartada porque acopla a latência e disponibilidade de sistemas de terceiros à operação crítica de mudança de status, podendo travar outros pedidos e sem caminho de rollback caso o cliente esteja indisponível (`[09:04] Bruno`).

### Fila dedicada (Redis Streams ou similar)

Publicar o evento em uma fila de mensageria externa (ex.: Redis Streams) e consumir dali.

- Descartada por exigir subir e operar infraestrutura adicional (ex.: Redis Cluster) que a equipe, sendo pequena, não tem capacidade de manter com baixo custo operacional (`[09:07] Larissa`, `[09:07] Diego`). O MySQL já existente resolve o problema sem nova dependência.

## Consequências

### Positivas

- Garantia transacional forte: é impossível um status mudar sem o evento correspondente ser registrado, e vice-versa, pois ambos ocorrem na mesma transação SQL.
- Nenhuma infraestrutura nova é introduzida; a solução reaproveita o MySQL e o Prisma já existentes no projeto.
- Observabilidade simples: o estado de cada evento (pendente/processando/falhou/entregue) fica auditável em uma única tabela relacional.

### Negativas

- Acopla a escrita de eventos de webhook ao mesmo banco transacional de pedidos, aumentando o volume de escrita na mesma instância MySQL que já sustenta a operação crítica de pedidos.
- Requer um processo de arquivamento/limpeza das linhas já entregues (mencionado como fora de escopo desta feature, `[09:08] Diego`) para a tabela não crescer indefinidamente.
- Não é um mecanismo de mensageria "nativo": depende inteiramente do worker de polling (ver ADR-002) para ter baixa latência, diferente de soluções com push nativo (ex.: `LISTEN/NOTIFY` do Postgres, que o MySQL não possui).

### Trade-off explícito

Trocamos a robustez e a escala horizontal de uma fila dedicada por simplicidade operacional e reuso de infraestrutura já existente. Para o volume atual e o tamanho da equipe, o ganho de não operar um novo serviço de mensageria supera a perda de recursos nativos de fila (ack distribuído, fan-out, particionamento nativo).
