# ADR-002: Worker dedicado em processo separado, com polling de 2 segundos

## Status

Aceito

## Data

Decisão tomada em reunião técnica de kickoff da feature (ver `TRANSCRICAO.md`). ADR registrado em 2026-10-05.

## Decisores

Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)

## Contexto

Com o padrão Outbox definido (ver [ADR-001](./ADR-001-outbox-no-mysql.md)), é necessário decidir como e onde os eventos pendentes na tabela `webhook_outbox` são lidos e processados.

`[09:09] Diego` propôs polling em loop: a cada 2 segundos, buscar os eventos pendentes mais antigos, processar e marcar como entregue. `[09:09] Bruno` questionou se seria possível usar triggers de banco para reagir de forma mais reativa. `[09:09] Diego` explicou que o MySQL não tem um mecanismo equivalente ao `LISTEN/NOTIFY` do PostgreSQL — um trigger de banco só executa SQL, não notifica um processo externo, e improvisar isso (escrever em arquivo, chamar um endpoint a partir do trigger) é uma solução frágil e fora do padrão. Polling de 2 segundos foi considerado suficiente, já que o requisito de negócio é "abaixo de 10 segundos" (`[09:02] Marcos`), confirmado em `[09:10] Marcos`.

`[09:11] Diego` levantou um segundo ponto, tratado na mesma decisão: o worker precisa rodar como **processo separado** da API, não dentro da mesma instância do servidor HTTP. Se rodasse no mesmo processo, um reinício da API (deploy, crash, restart) derrubaria o worker junto. `[09:11] Larissa` propôs um novo entry-point no projeto, análogo ao existente `src/server.ts`, criando um `src/worker.ts` com script próprio (`npm run worker`). `[09:11]-[09:12] Bruno/Diego` confirmaram que o worker usa o mesmo banco (`DATABASE_URL`) mas precisa de sua própria instância de `PrismaClient`, pois `PrismaClient` é por processo — mesmo banco, mesma stack, processo Node distinto.

Como consequência direta dessa decisão de execução, foi discutida a garantia de ordenação: com um único worker processando a outbox em ordem de `created_at`, a ordem de entrega ao cliente é preservada por `order_id` (`[09:12]-[09:13] Diego`). Essa limitação está registrada separadamente em [ADR-007](./ADR-007-ordenacao-por-order-id-sem-garantia-global.md).

## Decisão

Implementar o processamento da outbox como um **worker em processo Node.js separado** (`src/worker.ts`, novo entry-point no mesmo padrão do já existente `src/server.ts`), iniciado por um script dedicado (ex.: `npm run worker`). O worker instancia seu próprio `PrismaClient`, conectado à mesma `DATABASE_URL` da API, mas como instância independente (um `PrismaClient` por processo).

O worker opera em **polling com intervalo fixo de 2 segundos**: a cada ciclo, busca um lote pequeno dos eventos pendentes mais antigos na `webhook_outbox` (ordenados por `created_at`), processa cada um (envio HTTP ao endpoint do cliente) e atualiza o status do evento (entregue, falhou, ou mantém pendente para retry).

Para o estágio atual do produto, assume-se execução como **single-worker**: não há particionamento ou múltiplas instâncias concorrentes processando a outbox.

## Alternativas Consideradas

### Reagir via trigger de banco (equivalente a `LISTEN/NOTIFY`)

Usar um trigger no MySQL para notificar o worker assim que uma linha é inserida na outbox, reduzindo a latência mínima abaixo dos 2 segundos do polling.

- Descartada porque o MySQL não oferece um mecanismo nativo de notificação de processos externos a partir de triggers (ao contrário do `LISTEN/NOTIFY` do PostgreSQL); um trigger só executa SQL dentro do próprio banco. Viabilizar a notificação externa exigiria soluções improvisadas (escrita em arquivo, chamada HTTP a partir do banco), consideradas frágeis e fora do padrão pela equipe (`[09:09] Diego`).

### Worker executando dentro do mesmo processo da API

Rodar o loop de processamento da outbox como uma tarefa em background dentro do próprio processo do servidor HTTP (`src/server.ts`), evitando um segundo entry-point e um segundo processo para operar.

- Descartada porque acopla o ciclo de vida do worker ao da API: qualquer reinício, deploy ou crash do processo da API derrubaria também o processamento de eventos pendentes, sem necessidade (`[09:11] Diego`).

## Consequências

### Positivas

- Desacopla o ciclo de vida do processamento de webhooks do ciclo de vida da API: deploys, reinícios ou picos de carga na API não interrompem o processamento de eventos pendentes, e vice-versa.
- Latência previsível e simples de raciocinar: no pior caso, 2 segundos entre o commit da transação e o início do processamento, dentro da margem de "abaixo de 10 segundos" exigida pelos clientes B2B (`[09:02] Marcos`).
- Reaproveita a stack existente (Prisma, estrutura de módulos) sem introduzir um runtime ou paradigma novo de execução.

### Negativas

- Introduz um segundo processo a ser operado, monitorado e implantado (deploy, restart policy, health check), com disciplina operacional própria.
- Polling impõe uma latência mínima inerente (até 2 segundos) que uma arquitetura orientada a eventos nativos não teria.
- A garantia de ordenação de entrega depende de ser single-worker; qualquer evolução futura para múltiplos workers em paralelo exige redesenho (lock pessimista ou particionamento por `order_id`), como já antecipado em `[09:13] Diego`.

### Trade-off explícito

Aceitamos uma latência mínima de até 2 segundos e a operação de um processo adicional em troca de um desacoplamento simples e robusto entre API e processamento de webhooks, sem depender de recursos que o MySQL não oferece nativamente (notificação de processos externos).
