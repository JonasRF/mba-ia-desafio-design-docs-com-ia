# Tracker

Referência cruzada entre cada item registrado nos documentos ([PRD](./PRD.md), [RFC](./RFC.md), [FDD](./FDD.md) e [ADRs](./adrs/)) e a sua origem: uma fala da reunião (`TRANSCRICAO.md`) ou um arquivo do código base. Para itens confirmados por várias falas, a localização aponta a fala em que o item foi proposto ou decidido.

Cada seção abaixo cobre um documento. Todas as tabelas seguem o mesmo formato.

## PRD ([docs/PRD.md](./PRD.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-CTX-01 | docs/PRD.md | Contexto | Atlas, MaxDistribuição e Nova Cargo pediram notificação; hoje fazem polling em `GET /orders`, lento e caro | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Restrição | Atlas pode migrar para concorrente se a feature não sair até o fim do trimestre | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-03 | docs/PRD.md | Requisito Não Funcional | "Tempo real" para o cliente significa menos de 10 segundos | TRANSCRICAO | [09:02] Marcos |
| PRD-CTX-04 | docs/PRD.md | Restrição | Fluxo somente de saída, da plataforma para o cliente | TRANSCRICAO | [09:02] Marcos |
| PRD-CTX-05 | docs/PRD.md | Restrição | Mudança de status não pode ficar lenta nem falhar por causa do cliente | TRANSCRICAO | [09:04] Bruno |
| PRD-CTX-06 | docs/PRD.md | Restrição | Entrega apenas por API, sem interface visual nesta fase | TRANSCRICAO | [09:40] Larissa |
| PRD-PER-01 | docs/PRD.md | Persona | Usuário autenticado no OMS que representa o cliente gerencia os webhooks pela API | TRANSCRICAO | [09:32] Marcos |
| PRD-PER-02 | docs/PRD.md | Persona | Administrador com role `ADMIN` reprocessa notificações | TRANSCRICAO | [09:36] Sofia |
| PRD-PER-03 | docs/PRD.md | Persona | Roles `ADMIN` e `OPERATOR` já existem no modelo de usuários | CODIGO | prisma/schema.prisma |
| PRD-CEN-01 | docs/PRD.md | Cenário de uso | Cliente assina só `SHIPPED` e `DELIVERED` e não recebe os demais status | TRANSCRICAO | [09:33] Marcos |
| PRD-CEN-02 | docs/PRD.md | Cenário de uso | Cliente confere a assinatura com a secret e descarta o que não confere | TRANSCRICAO | [09:19] Sofia |
| PRD-CEN-03 | docs/PRD.md | Cenário de uso | Manutenção planejada de 2 horas é atravessada pelas retentativas | TRANSCRICAO | [09:16] Diego |
| PRD-CEN-04 | docs/PRD.md | Cenário de uso | Cliente reconhece o identificador do evento e ignora a duplicata | TRANSCRICAO | [09:25] Diego |
| PRD-CEN-05 | docs/PRD.md | Cenário de uso | Secret vazada em log do cliente é trocada pela API com 24h de carência | TRANSCRICAO | [09:22] Diego |
| PRD-CEN-06 | docs/PRD.md | Cenário de uso | Cliente consulta as últimas entregas para investigar problema | TRANSCRICAO | [09:34] Marcos |
| PRD-CEN-07 | docs/PRD.md | Cenário de uso | Administrador reprocessa a dead letter e a ação fica registrada | TRANSCRICAO | [09:36] Sofia |
| PRD-OBJ-01 | docs/PRD.md | Objetivo / Métrica | O1: primeira tentativa de entrega em menos de 10 segundos | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | docs/PRD.md | Objetivo / Métrica | O2: em produção até o fim de novembro, em 3 sprints, com revisão de segurança | TRANSCRICAO | [09:46] Larissa |
| PRD-OBJ-03 | docs/PRD.md | Objetivo / Métrica | O3: 3 de 3 clientes solicitantes com webhook ativo | TRANSCRICAO | [09:00] Marcos |
| PRD-OBJ-04 | docs/PRD.md | Objetivo / Métrica | O4: cerca de 15 horas de indisponibilidade cobertas por retentativas | TRANSCRICAO | [09:17] Diego |
| PRD-OBJ-05 | docs/PRD.md | Objetivo / Métrica | O5: zero mudanças de status commitadas sem evento registrado | TRANSCRICAO | [09:40] Bruno |
| PRD-OOS-01 | docs/PRD.md | Fora de escopo | Aviso por email quando o webhook falha: adiado para a próxima fase | TRANSCRICAO | [09:37] Larissa |
| PRD-OOS-02 | docs/PRD.md | Fora de escopo | Dashboard visual: projeto separado do time de frontend | TRANSCRICAO | [09:40] Larissa |
| PRD-OOS-03 | docs/PRD.md | Fora de escopo | Rate limiting de envio: observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| PRD-OOS-04 | docs/PRD.md | Fora de escopo | Webhooks de entrada descartados | TRANSCRICAO | [09:02] Marcos |
| PRD-OOS-05 | docs/PRD.md | Fora de escopo | Garantia exactly-once descartada | TRANSCRICAO | [09:25] Diego |
| PRD-OOS-06 | docs/PRD.md | Fora de escopo | Garantia de ordem global entre pedidos descartada | TRANSCRICAO | [09:14] Marcos |
| PRD-OOS-07 | docs/PRD.md | Fora de escopo | Múltiplos workers em paralelo adiados ("problema do futuro") | TRANSCRICAO | [09:13] Diego |
| PRD-OOS-08 | docs/PRD.md | Fora de escopo | Arquivamento de eventos entregues fora desta feature | TRANSCRICAO | [09:08] Diego |
| PRD-OOS-09 | docs/PRD.md | Fora de escopo | Permissões mais restritas no CRUD adiadas | TRANSCRICAO | [09:37] Sofia |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cadastrar webhook com cliente, URL e lista de status | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-01b | docs/PRD.md | Restrição | `customer_id` informado na requisição, não deduzido do JWT | TRANSCRICAO | [09:32] Larissa |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Secret gerada pela plataforma e devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Editar webhook (PATCH) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Remover webhook (DELETE) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Listar webhooks de um cliente (GET) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Filtro de status por webhook, aplicado na inserção da outbox | TRANSCRICAO | [09:34] Bruno |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Evento registrado atomicamente com a mudança de status | TRANSCRICAO | [09:40] Bruno |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Payload JSON com `event_id`, `event_type`, `timestamp`, ids, status e `total_cents`, sem itens | TRANSCRICAO | [09:43] Diego |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Notificação reflete o pedido no momento da mudança (snapshot) | TRANSCRICAO | [09:52] Larissa |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `Content-Type` | TRANSCRICAO | [09:44] Diego |
| PRD-FR-10b | docs/PRD.md | Requisito Funcional | Header `X-Webhook-Id` com o id do endpoint | TRANSCRICAO | [09:44] Sofia |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | Assinatura HMAC-SHA256 sobre o corpo com a secret do webhook | TRANSCRICAO | [09:20] Sofia |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | Rotação de secret pela API, anterior válida por 24h | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-13 | docs/PRD.md | Requisito Funcional | Retry automático: 5 tentativas em 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-14 | docs/PRD.md | Requisito Funcional | Esgotadas as tentativas, evento vai para dead letter com payload, motivo e data | TRANSCRICAO | [09:18] Diego |
| PRD-FR-15 | docs/PRD.md | Requisito Funcional | Administrador reprocessa manualmente evento da dead letter | TRANSCRICAO | [09:35] Diego |
| PRD-FR-16 | docs/PRD.md | Requisito Funcional | Histórico das últimas 100 entregas com resultado, payload, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Latência < 10s; verificação de pendentes a cada 2s | TRANSCRICAO | [09:10] Larissa |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Entrega at-least-once, duplicatas possíveis | TRANSCRICAO | [09:24] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Cliente lento ou fora do ar não afeta mudanças de status | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Entregas em processo separado da API | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Ordem garantida só por pedido, sem ordem global | TRANSCRICAO | [09:13] Larissa |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | Timeout de 10s por chamada; depois disso conta como falha | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-07 | docs/PRD.md | Requisito Não Funcional | Só URLs `https`; `http` recusado com erro de validação | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-08 | docs/PRD.md | Requisito Não Funcional | Secret própria por webhook; sem secret global | TRANSCRICAO | [09:21] Sofia |
| PRD-NFR-09 | docs/PRD.md | Requisito Não Funcional | Replay exige `ADMIN`; demais endpoints só autenticação | TRANSCRICAO | [09:36] Larissa |
| PRD-NFR-10 | docs/PRD.md | Requisito Não Funcional | Todo replay registra quem executou (auditoria) | TRANSCRICAO | [09:36] Sofia |
| PRD-NFR-11 | docs/PRD.md | Requisito Não Funcional | Payload acima de 64KB gera erro, sem truncar | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-12 | docs/PRD.md | Requisito Não Funcional | Nenhuma infraestrutura nova; MySQL e stack existentes | TRANSCRICAO | [09:07] Diego |
| PRD-NFR-13 | docs/PRD.md | Requisito Não Funcional | Seguir padrões do projeto: módulos, `AppError`, Pino, Zod, prefixo `WEBHOOK_` | TRANSCRICAO | [09:30] Larissa |
| PRD-DEC-01 | docs/PRD.md | Trade-off | Payload enxuto sem itens; detalhes via `GET /orders/:id` | TRANSCRICAO | [09:43] Diego |
| PRD-DEP-01 | docs/PRD.md | Dependência | Revisão de segurança de HMAC e secret com 2 dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-02 | docs/PRD.md | Dependência | Documentação no portal do desenvolvedor (integração, assinatura, dedup) | TRANSCRICAO | [09:26] Marcos |
| PRD-DEP-03 | docs/PRD.md | Dependência | Cliente expõe endpoint `https`, valida assinatura, deduplica e responde em 10s | TRANSCRICAO | [09:42] Diego |
| PRD-DEP-04 | docs/PRD.md | Dependência | Usuários que representam o cliente, autenticados com JWT do sistema | TRANSCRICAO | [09:32] Marcos |
| PRD-DEP-05 | docs/PRD.md | Dependência | Mudança de status em `order.service.ts` passa a registrar o evento | CODIGO | src/modules/orders/order.service.ts |
| PRD-DEP-06 | docs/PRD.md | Dependência | Controle de role existente (`requireRole`) | CODIGO | src/middlewares/auth.middleware.ts |
| PRD-DEP-07 | docs/PRD.md | Dependência | MySQL compartilhado entre API e processo de entregas | TRANSCRICAO | [09:11] Bruno |
| PRD-DEP-08 | docs/PRD.md | Dependência | Sessão de revisão do design com Bruno e Diego antes de codar | TRANSCRICAO | [09:50] Larissa |
| PRD-DEP-09 | docs/PRD.md | Dependência | Confirmação do prazo com a Atlas | TRANSCRICAO | [09:47] Marcos |
| PRD-RISK-01 | docs/PRD.md | Risco | Prazo de 3 sprints estoura e Atlas migra | TRANSCRICAO | [09:45] Marcos |
| PRD-RISK-02 | docs/PRD.md | Risco | Cliente não deduplica e processa a notificação duas vezes | TRANSCRICAO | [09:25] Sofia |
| PRD-RISK-03 | docs/PRD.md | Risco | Falha do webhook não percebida pelo cliente, sem aviso por email | TRANSCRICAO | [09:37] Marcos |
| PRD-RISK-04 | docs/PRD.md | Risco | Cliente fora do ar por mais de ~15h e eventos vão para a dead letter | TRANSCRICAO | [09:17] Marcos |
| PRD-RISK-05 | docs/PRD.md | Risco | Vazamento de secret do lado do cliente | TRANSCRICAO | [09:22] Diego |
| PRD-RISK-06 | docs/PRD.md | Risco | Rajada de 50 mudanças em um minuto sobrecarrega o cliente | TRANSCRICAO | [09:38] Diego |
| PRD-RISK-07 | docs/PRD.md | Risco | Problema no registro do evento impede mudanças de status | TRANSCRICAO | [09:40] Bruno |
| PRD-TEST-01 | docs/PRD.md | Estratégia de teste | Testes com Vitest e Supertest e banco real, no molde dos atuais | CODIGO | tests/orders.test.ts |
| PRD-TEST-02 | docs/PRD.md | Estratégia de teste | Regressão: suíte de autenticação continua passando | CODIGO | tests/auth.test.ts |
| PRD-TEST-03 | docs/PRD.md | Estratégia de teste | Testes ponta a ponta previstos na estimativa junto com a integração em pedidos | TRANSCRICAO | [09:46] Larissa |
| PRD-TEST-04 | docs/PRD.md | Estratégia de teste | Volume de notificações por webhook como sinal para decidir rate limiting | TRANSCRICAO | [09:39] Diego |

## RFC ([docs/RFC.md](./RFC.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| RFC-CTX-01 | docs/RFC.md | Contexto | `changeStatus` atualiza `orders`, `order_status_history` e `stock_quantity` em uma transação | CODIGO | src/modules/orders/order.service.ts |
| RFC-CTX-02 | docs/RFC.md | Restrição | Webhook somente outbound | TRANSCRICAO | [09:03] Sofia |
| RFC-PROP-01 | docs/RFC.md | Decisão | Evento gravado em `webhook_outbox` na mesma transação da mudança de status | TRANSCRICAO | [09:06] Diego |
| RFC-PROP-02 | docs/RFC.md | Decisão | Evento só é inserido se algum webhook ativo assina o status | TRANSCRICAO | [09:34] Bruno |
| RFC-PROP-03 | docs/RFC.md | Decisão | Worker em processo separado lê pendentes a cada 2 segundos | TRANSCRICAO | [09:09] Diego |
| RFC-PROP-04 | docs/RFC.md | Decisão | Single-worker preserva ordem por pedido | TRANSCRICAO | [09:12] Diego |
| RFC-PROP-05 | docs/RFC.md | Decisão | DLQ `webhook_dead_letter` com replay manual por `ADMIN` | TRANSCRICAO | [09:18] Diego |
| RFC-PROP-06 | docs/RFC.md | Requisito Não Funcional | Só URLs `https`; payload acima de 64KB gera erro | TRANSCRICAO | [09:23] Sofia |
| RFC-PROP-07 | docs/RFC.md | Decisão | Feature como módulo `src/modules/webhooks` no molde dos demais | TRANSCRICAO | [09:27] Bruno |
| RFC-PROP-08 | docs/RFC.md | Requisito Funcional | Superfície de API: CRUD, histórico de entregas e replay de DLQ | TRANSCRICAO | [09:31] Marcos |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | Disparo síncrono dentro de `changeStatus` | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Fila dedicada (Redis Streams): infra nova, overengineering | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger de banco: MySQL não notifica processo externo | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Retry indefinido ou teto de 3 tentativas | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-05 | docs/RFC.md | Alternativa descartada | Exactly-once: coordenação dos dois lados, complexidade alta | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-06 | docs/RFC.md | Alternativa descartada | Secret global: vazamento de uma compromete todos | TRANSCRICAO | [09:21] Sofia |
| RFC-OQ-01 | docs/RFC.md | Questão em aberto | Rate limiting de saída: observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| RFC-OQ-02 | docs/RFC.md | Questão em aberto | Aviso ao cliente por email quando o webhook falha | TRANSCRICAO | [09:37] Larissa |
| RFC-OQ-03 | docs/RFC.md | Questão em aberto | Escala além de single-worker (particionar por `order_id` ou lock) | TRANSCRICAO | [09:13] Diego |
| RFC-OQ-04 | docs/RFC.md | Questão em aberto | Retenção/arquivamento da outbox "depois de 30 dias ou assim" | TRANSCRICAO | [09:08] Diego |
| RFC-OQ-05 | docs/RFC.md | Questão em aberto | Endurecimento da autorização do CRUD de webhooks | TRANSCRICAO | [09:37] Sofia |
| RFC-OQ-06 | docs/RFC.md | Questão em aberto | `customer_id` no body ou no path (só ficou definido que não vem do JWT) | TRANSCRICAO | [09:32] Larissa |
| RFC-IMP-01 | docs/RFC.md | Impacto | `changeStatus` é a única alteração em código crítico | TRANSCRICAO | [09:40] Bruno |
| RFC-IMP-02 | docs/RFC.md | Impacto | Novas tabelas na mesma instância MySQL | TRANSCRICAO | [09:07] Diego |
| RFC-IMP-03 | docs/RFC.md | Impacto | Segundo processo a implantar e monitorar | TRANSCRICAO | [09:11] Diego |
| RFC-IMP-04 | docs/RFC.md | Impacto | Clientes precisam validar HMAC e deduplicar; documentado no portal | TRANSCRICAO | [09:26] Marcos |
| RFC-IMP-05 | docs/RFC.md | Impacto | Prazo de três sprints com revisão de segurança incluída | TRANSCRICAO | [09:47] Larissa |
| RFC-RISK-01 | docs/RFC.md | Risco | Falha na inserção da outbox causa rollback da mudança de status | TRANSCRICAO | [09:40] Bruno |
| RFC-RISK-02 | docs/RFC.md | Risco | Cliente não deduplica por `X-Event-Id` | TRANSCRICAO | [09:25] Sofia |
| RFC-RISK-03 | docs/RFC.md | Risco | Worker único parado ou sobrecarregado atrasa todas as entregas | TRANSCRICAO | [09:12] Diego |
| RFC-RISK-04 | docs/RFC.md | Risco | Crescimento da outbox degrada a leitura do worker | TRANSCRICAO | [09:07] Bruno |
| RFC-RISK-05 | docs/RFC.md | Risco | Vazamento de secret do lado do cliente | TRANSCRICAO | [09:22] Diego |
| RFC-RISK-06 | docs/RFC.md | Risco | Rajada de eventos sobrecarrega o endpoint do cliente | TRANSCRICAO | [09:38] Diego |
| RFC-RISK-07 | docs/RFC.md | Risco | Eventos na DLQ dependem de ação manual | TRANSCRICAO | [09:18] Diego |
| RFC-RISK-08 | docs/RFC.md | Risco | Prazo estoura e compromete o compromisso com a Atlas | TRANSCRICAO | [09:45] Marcos |

## FDD ([docs/FDD.md](./FDD.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| FDD-CTX-01 | docs/FDD.md | Contexto | `changeStatus` roda em `prisma.$transaction` com update, histórico e estoque | CODIGO | src/modules/orders/order.service.ts |
| FDD-CTX-02 | docs/FDD.md | Restrição | Sem chamada HTTP dentro da transação de status | TRANSCRICAO | [09:04] Bruno |
| FDD-DADOS-01 | docs/FDD.md | Restrição | Chaves primárias UUID em `@db.Char(36)`, como o restante do schema | CODIGO | prisma/schema.prisma |
| FDD-DADOS-02 | docs/FDD.md | Decisão | Outbox usa UUID, não auto incremento | TRANSCRICAO | [09:51] Larissa |
| FDD-DADOS-03 | docs/FDD.md | Modelo de dados | `webhook_endpoints` guarda `url`, `secret`, `customerId` e `active` | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-04 | docs/FDD.md | Modelo de dados | `events`: lista de status que o webhook quer ouvir | TRANSCRICAO | [09:33] Marcos |
| FDD-DADOS-05 | docs/FDD.md | Modelo de dados | Enum `PENDING`, `PROCESSING`, `FAILED`, `DELIVERED` na outbox | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-06 | docs/FDD.md | Modelo de dados | Índices da outbox em status e `createdAt` | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-07 | docs/FDD.md | Modelo de dados | `webhook_deliveries`: sucesso, status, resposta e duração por tentativa | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-08 | docs/FDD.md | Modelo de dados | `webhook_dead_letter` com payload, motivo e data da falha | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-09 | docs/FDD.md | Restrição | `outbox.orderId` sem FK, pois `OrderService.delete` apaga pedidos `PENDING`/`CANCELLED` | CODIGO | src/modules/orders/order.service.ts |
| FDD-DADOS-10 | docs/FDD.md | Decisão | `id` da outbox gerado na aplicação com o pacote `uuid` já instalado | CODIGO | package.json |
| FDD-FLUXO-01 | docs/FDD.md | Decisão | `publishWebhookEvent(tx, order, fromStatus, toStatus)` recebe o tx da transação | TRANSCRICAO | [09:41] Bruno |
| FDD-FLUXO-02 | docs/FDD.md | Regra de negócio | Filtra endpoints por `events`; sem assinante, não insere | TRANSCRICAO | [09:34] Bruno |
| FDD-FLUXO-03 | docs/FDD.md | Regra de negócio | Payload acima de 64KB lança `WEBHOOK_PAYLOAD_TOO_LARGE` | TRANSCRICAO | [09:23] Sofia |
| FDD-FLUXO-04 | docs/FDD.md | Regra de negócio | Erro em `publishWebhookEvent` desfaz a transação inteira | TRANSCRICAO | [09:40] Bruno |
| FDD-FLUXO-05 | docs/FDD.md | Restrição | `OrderService.create` grava histórico `null → PENDING` e não publica evento | CODIGO | src/modules/orders/order.service.ts |
| FDD-FLUXO-06 | docs/FDD.md | Decisão | Entry-point `src/worker.ts`; lógica em `webhook.processor.ts` | TRANSCRICAO | [09:28] Bruno |
| FDD-FLUXO-07 | docs/FDD.md | Regra de negócio | Worker lê lote pequeno dos pendentes mais antigos a cada 2s | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-08 | docs/FDD.md | Regra de negócio | Seleção preserva ordem por pedido (`webhookId`, `orderId`) | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-09 | docs/FDD.md | Regra de negócio | `POST` com timeout de 10 segundos | TRANSCRICAO | [09:42] Diego |
| FDD-FLUXO-10 | docs/FDD.md | Regra de negócio | Worker envia o payload armazenado, sem reler o pedido | TRANSCRICAO | [09:52] Larissa |
| FDD-FLUXO-11 | docs/FDD.md | Regra de negócio | Tabela de retry 1m, 5m, 30m, 2h, 12h | TRANSCRICAO | [09:17] Diego |
| FDD-FLUXO-12 | docs/FDD.md | Regra de negócio | Replay recoloca o evento na outbox como `PENDING` | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-13 | docs/FDD.md | Regra de negócio | Replay mantém o mesmo `event_id` para preservar a deduplicação | TRANSCRICAO | [09:25] Diego |
| FDD-FLUXO-14 | docs/FDD.md | Requisito Não Funcional | Replay grava `replayedById` e emite log de auditoria | TRANSCRICAO | [09:36] Sofia |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | Base `/api/v1` | CODIGO | src/app.ts |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | Formato de erro `{ error: { code, message, details } }` | CODIGO | src/middlewares/error.middleware.ts |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | `customerId` no body e na query, como nos schemas de pedidos | CODIGO | src/modules/orders/order.schemas.ts |
| FDD-CONTRATO-04 | docs/FDD.md | Restrição | `customerId` não vem do JWT | TRANSCRICAO | [09:32] Larissa |
| FDD-CONTRATO-05 | docs/FDD.md | Restrição | Qualquer role autenticada gerencia webhooks nesta fase | TRANSCRICAO | [09:37] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | `POST /webhooks` devolve a secret gerada na criação | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | `GET /webhooks` paginado no formato de `paginated()` | CODIGO | src/shared/http/response.ts |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | `PATCH /webhooks/:id` e `DELETE /webhooks/:id` | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-09 | docs/FDD.md | Contrato | `POST /webhooks/:id/rotate-secret` com carência de 24h | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-10 | docs/FDD.md | Contrato | `GET /webhooks/:id/deliveries`, até 100 por página | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-11 | docs/FDD.md | Contrato | `POST /admin/webhooks/dead-letter/:id/replay` | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-12 | docs/FDD.md | Restrição | Replay protegido por `requireRole('ADMIN')` | TRANSCRICAO | [09:36] Larissa |
| FDD-CONTRATO-13 | docs/FDD.md | Contrato | `OPERATOR` recebe `403 FORBIDDEN` no replay (roles e `requireRole` existentes) | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-CONTRATO-14 | docs/FDD.md | Contrato | Headers `Content-Type`, `X-Event-Id`, `X-Timestamp`, `X-Signature` | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-15 | docs/FDD.md | Contrato | Header `X-Webhook-Id` | TRANSCRICAO | [09:44] Sofia |
| FDD-CONTRATO-16 | docs/FDD.md | Contrato | `X-Signature` com HMAC-SHA256 do corpo | TRANSCRICAO | [09:20] Sofia |
| FDD-CONTRATO-17 | docs/FDD.md | Contrato | Corpo snake_case com `event_type: order.status_changed`, sem `items` | TRANSCRICAO | [09:43] Diego |
| FDD-ERRO-01 | docs/FDD.md | Padrão de código | Novas classes no molde de `InsufficientStockError` e `InvalidStatusTransitionError` | CODIGO | src/shared/errors/http-errors.ts |
| FDD-ERRO-02 | docs/FDD.md | Decisão | Códigos de erro com prefixo `WEBHOOK_` | TRANSCRICAO | [09:29] Larissa |
| FDD-ERRO-03 | docs/FDD.md | Decisão | Códigos citados: `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED` | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-04 | docs/FDD.md | Requisito Não Funcional | URL sem `https` gera `WEBHOOK_INVALID_URL` | TRANSCRICAO | [09:23] Sofia |
| FDD-ERRO-05 | docs/FDD.md | Restrição | `NotFoundError` fixa o código `NOT_FOUND`; erros 404 de webhook estendem `AppError` | CODIGO | src/shared/errors/http-errors.ts |
| FDD-ERRO-06 | docs/FDD.md | Restrição | Base `AppError` com `statusCode` e `errorCode` | CODIGO | src/shared/errors/app-error.ts |
| FDD-ERRO-07 | docs/FDD.md | Restrição | Middleware `validate` converte falha de Zod em `VALIDATION_ERROR` | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-ERRO-08 | docs/FDD.md | Decisão | `errorMiddleware` trata os novos erros sem alteração | TRANSCRICAO | [09:29] Bruno |
| FDD-ERRO-09 | docs/FDD.md | Regra de negócio | `WEBHOOK_DELIVERY_TIMEOUT` após 10s gera retry | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-10 | docs/FDD.md | Regra de negócio | `WEBHOOK_MAX_ATTEMPTS_EXCEEDED` move para a DLQ | TRANSCRICAO | [09:15] Diego |
| FDD-RES-01 | docs/FDD.md | Decisão | Timeout via `fetch` nativo; projeto exige Node `>=20` | CODIGO | package.json |
| FDD-RES-02 | docs/FDD.md | Requisito Não Funcional | Isolamento: queda da API não interrompe entregas | TRANSCRICAO | [09:11] Diego |
| FDD-RES-03 | docs/FDD.md | Padrão de código | Encerramento gracioso em `SIGINT`/`SIGTERM` com `prisma.$disconnect()` | CODIGO | src/server.ts |
| FDD-RES-04 | docs/FDD.md | Padrão de código | Parâmetros `WEBHOOK_*` no schema Zod de ambiente | CODIGO | src/config/env.ts |
| FDD-RES-05 | docs/FDD.md | Restrição | Sem fallback automático: email fora do escopo | TRANSCRICAO | [09:37] Larissa |
| FDD-OBS-01 | docs/FDD.md | Restrição | Observabilidade sobre Pino, sem biblioteca nova de métricas ou tracing | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | docs/FDD.md | Padrão de código | Logger Pino existente com `redactPaths` | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-03 | docs/FDD.md | Requisito Não Funcional | Secrets e assinaturas nunca logadas (caso de vazamento em log) | TRANSCRICAO | [09:22] Diego |
| FDD-OBS-04 | docs/FDD.md | Requisito Não Funcional | Log `webhook_dead_letter_replayed` com `userId` | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-05 | docs/FDD.md | Métrica | Idade do pendente mais antigo como indicador do requisito de 10s | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-06 | docs/FDD.md | Métrica | Entregas por webhook como sinal para rate limiting | TRANSCRICAO | [09:39] Diego |
| FDD-OBS-07 | docs/FDD.md | Tracing | Correlação pelo `requestId` já gerado na API | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-INT-01 | docs/FDD.md | Integração | Chamada a `publishWebhookEvent` após `tx.orderStatusHistory.create` | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Decisão | Sem injetar repository no `OrderService`; função pura recebendo o tx | TRANSCRICAO | [09:41] Diego |
| FDD-INT-03 | docs/FDD.md | Integração | `buildControllers` instancia repository, service e controller de webhooks | CODIGO | src/app.ts |
| FDD-INT-04 | docs/FDD.md | Integração | `buildApiRouter` monta `/webhooks` e `/admin/webhooks` | CODIGO | src/routes/index.ts |
| FDD-INT-05 | docs/FDD.md | Integração | `authenticate` aplicado com `router.use`, como nas rotas de pedidos | CODIGO | src/modules/orders/order.routes.ts |
| FDD-INT-06 | docs/FDD.md | Decisão | Worker com `PrismaClient` próprio, mesma `DATABASE_URL` | TRANSCRICAO | [09:30] Bruno |
| FDD-INT-07 | docs/FDD.md | Integração | Worker importa `prisma` de `database.ts` | CODIGO | src/config/database.ts |
| FDD-INT-08 | docs/FDD.md | Decisão | Script `npm run worker` para o novo entry-point | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-09 | docs/FDD.md | Integração | Limpeza das novas tabelas antes de `customer.deleteMany()` | CODIGO | tests/setup.ts |
| FDD-DEP-01 | docs/FDD.md | Restrição | Nenhuma dependência nova | TRANSCRICAO | [09:29] Bruno |
| FDD-DEP-02 | docs/FDD.md | Restrição | MySQL 8.0 via Docker Compose | CODIGO | docker-compose.yml |
| FDD-DEP-03 | docs/FDD.md | Dependência | Revisão de segurança de HMAC e secret antes do deploy | TRANSCRICAO | [09:46] Sofia |
| FDD-DEP-04 | docs/FDD.md | Dependência | Documentação no portal sobre assinatura e deduplicação | TRANSCRICAO | [09:26] Marcos |
| FDD-CA-01 | docs/FDD.md | Critério de aceite | Status assinado cria linha `PENDING` por endpoint assinante | TRANSCRICAO | [09:34] Bruno |
| FDD-CA-03 | docs/FDD.md | Critério de aceite | Falha na outbox mantém status, histórico e estoque inalterados | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-04 | docs/FDD.md | Critério de aceite | Primeira tentativa em menos de 10s após o commit | TRANSCRICAO | [09:10] Larissa |
| FDD-CA-06 | docs/FDD.md | Critério de aceite | Falha, timeout ou erro de rede agenda retry pela tabela | TRANSCRICAO | [09:17] Diego |
| FDD-CA-09 | docs/FDD.md | Critério de aceite | Secret nova e anterior válidas por 24h após rotação | TRANSCRICAO | [09:21] Sofia |
| FDD-CA-10 | docs/FDD.md | Critério de aceite | Mesmo `X-Event-Id` em todas as tentativas | TRANSCRICAO | [09:25] Diego |
| FDD-CA-12 | docs/FDD.md | Critério de aceite | URL `http://` recusada com `400 WEBHOOK_INVALID_URL` | TRANSCRICAO | [09:23] Sofia |
| FDD-CA-13 | docs/FDD.md | Critério de aceite | Replay por `OPERATOR` retorna 403; por `ADMIN` gera log de auditoria | TRANSCRICAO | [09:36] Sofia |
| FDD-CA-16 | docs/FDD.md | Critério de aceite | Suíte existente de pedidos e auth passa sem alteração | CODIGO | tests/orders.test.ts |
| FDD-PLAN-01 | docs/FDD.md | Plano | Ordem de implementação: outbox/DLQ, worker/retry, CRUD, integração, testes, revisão | TRANSCRICAO | [09:46] Larissa |
| FDD-RISK-01 | docs/FDD.md | Risco | Erro em `publishWebhookEvent` bloqueia mudanças de status | TRANSCRICAO | [09:40] Bruno |
| FDD-RISK-02 | docs/FDD.md | Risco | Evento em retry retém os seguintes do mesmo pedido | TRANSCRICAO | [09:12] Diego |
| FDD-RISK-03 | docs/FDD.md | Risco | Secret exposta em banco ou log | TRANSCRICAO | [09:22] Diego |
| FDD-RISK-04 | docs/FDD.md | Risco | Cliente não deduplica e processa o evento duas vezes | TRANSCRICAO | [09:25] Sofia |
| FDD-RISK-05 | docs/FDD.md | Risco | Outbox cresce e degrada a seleção do worker | TRANSCRICAO | [09:07] Bruno |
| FDD-RISK-06 | docs/FDD.md | Risco | Endpoint lento atrasa outros clientes; mitigado pelo timeout de 10s | TRANSCRICAO | [09:42] Diego |
| FDD-RISK-07 | docs/FDD.md | Risco | Rajada de eventos sobrecarrega o cliente | TRANSCRICAO | [09:38] Diego |

## ADRs

### ADR-001: Padrão Outbox no MySQL para publicação de eventos de webhook ([ADR-001-outbox-no-mysql.md](./adrs/ADR-001-outbox-no-mysql.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Padrão Outbox no MySQL existente | TRANSCRICAO | [09:08] Larissa |
| ADR-001-CTX-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | `changeStatus` já executa update, histórico e estoque em `this.prisma.$transaction` | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-CTX-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | Pergunta central: síncrono ou fila/outbox | TRANSCRICAO | [09:03] Larissa |
| ADR-001-DEC-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Garantia: commit registra o evento, rollback descarta junto | TRANSCRICAO | [09:06] Diego |
| ADR-001-DEC-03 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Índice em status e `created_at` para leitura dos pendentes | TRANSCRICAO | [09:08] Diego |
| ADR-001-ALT-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Disparo síncrono: cliente lento trava outros pedidos, sem rollback possível | TRANSCRICAO | [09:04] Bruno |
| ADR-001-ALT-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Redis Streams exigiria infraestrutura nova | TRANSCRICAO | [09:07] Larissa |
| ADR-001-TO-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Trade-off | Simplicidade operacional no lugar de fila dedicada; time pequeno | TRANSCRICAO | [09:07] Diego |
| ADR-001-CONS-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência | Necessidade futura de arquivar linhas entregues | TRANSCRICAO | [09:08] Diego |

### ADR-002: Worker dedicado em processo separado, com polling de 2 segundos ([ADR-002-worker-dedicado-em-polling.md](./adrs/ADR-002-worker-dedicado-em-polling.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-002 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Decisão | Worker em polling de 2 segundos | TRANSCRICAO | [09:10] Larissa |
| ADR-002-DEC-02 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Decisão | Worker em processo separado da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-DEC-03 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Decisão | Novo entry-point `src/worker.ts` análogo a `src/server.ts` | TRANSCRICAO | [09:11] Larissa |
| ADR-002-DEC-04 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Decisão | `PrismaClient` próprio por processo, mesma `DATABASE_URL` | TRANSCRICAO | [09:30] Bruno |
| ADR-002-CTX-01 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Contexto | Entry-point de referência `src/server.ts` | CODIGO | src/server.ts |
| ADR-002-CTX-02 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Contexto | 2 segundos aceito pelo produto | TRANSCRICAO | [09:10] Marcos |
| ADR-002-ALT-01 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Alternativa descartada | Trigger de banco para reagir à inserção | TRANSCRICAO | [09:09] Bruno |
| ADR-002-ALT-02 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Alternativa descartada | Worker no mesmo processo da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-TO-01 | docs/adrs/ADR-002-worker-dedicado-em-polling.md | Trade-off | MySQL sem `LISTEN/NOTIFY`; aceita latência mínima de até 2s | TRANSCRICAO | [09:09] Diego |

### ADR-003: Retry com backoff exponencial e Dead Letter Queue em tabela separada ([ADR-003-retry-com-backoff-e-dead-letter-queue.md](./adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md | Decisão | 5 tentativas com backoff 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Larissa |
| ADR-003-DEC-02 | docs/adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md | Decisão | DLQ em tabela separada `webhook_dead_letter` | TRANSCRICAO | [09:18] Diego |
| ADR-003-DEC-03 | docs/adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md | Decisão | Replay manual via endpoint admin | TRANSCRICAO | [09:18] Diego |
| ADR-003-CTX-01 | docs/adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md | Contexto | Cliente já teve indisponibilidade de 2h em manutenção planejada | TRANSCRICAO | [09:16] Diego |
| ADR-003-ALT-01 | docs/adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md | Alternativa descartada | Retry indefinido deixa evento pendurado para sempre | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | docs/adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md | Alternativa descartada | Teto de 3 tentativas, "mais agressivo" | TRANSCRICAO | [09:16] Bruno |
| ADR-003-ALT-03 | docs/adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md | Alternativa descartada | Marcar `failed` na própria outbox | TRANSCRICAO | [09:17] Larissa |
| ADR-003-TO-01 | docs/adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md | Trade-off | ~15h até falha permanente é aceitável para o produto | TRANSCRICAO | [09:17] Marcos |

### ADR-004: Autenticação dos webhooks via HMAC-SHA256 com secret por endpoint ([ADR-004-hmac-sha256-com-secret-por-endpoint.md](./adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | HMAC-SHA256 sobre o corpo, secret por endpoint, rotação com 24h | TRANSCRICAO | [09:22] Sofia |
| ADR-004-CTX-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Contexto | Cliente precisa validar origem e integridade do payload | TRANSCRICAO | [09:19] Sofia |
| ADR-004-DEC-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | Algoritmo SHA-256, padrão de mercado | TRANSCRICAO | [09:20] Sofia |
| ADR-004-DEC-03 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | Secret rotacionável com grace period de 24h | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CTX-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Contexto | Incidente real de secret vazada em log de cliente | TRANSCRICAO | [09:22] Diego |
| ADR-004-ALT-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Alternativa descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ALT-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Alternativa descartada | Rotação imediata, sem carência (implícita na decisão) | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CONS-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Consequência | Validação de URL `https` no cadastro | TRANSCRICAO | [09:23] Sofia |

### ADR-005: Garantia de entrega at-least-once com deduplicação via X-Event-Id ([ADR-005-garantia-at-least-once-com-event-id.md](./adrs/ADR-005-garantia-at-least-once-com-event-id.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-005 | docs/adrs/ADR-005-garantia-at-least-once-com-event-id.md | Decisão | Entrega at-least-once com `X-Event-Id` | TRANSCRICAO | [09:26] Larissa |
| ADR-005-DEC-02 | docs/adrs/ADR-005-garantia-at-least-once-com-event-id.md | Decisão | UUID gerado na entrada da outbox, enviado em `X-Event-Id` | TRANSCRICAO | [09:25] Diego |
| ADR-005-CTX-01 | docs/adrs/ADR-005-garantia-at-least-once-com-event-id.md | Contexto | Cliente pode receber o mesmo evento duas vezes | TRANSCRICAO | [09:24] Diego |
| ADR-005-ALT-01 | docs/adrs/ADR-005-garantia-at-least-once-com-event-id.md | Alternativa descartada | Exactly-once; Stripe e GitHub usam at-least-once | TRANSCRICAO | [09:25] Diego |
| ADR-005-TO-01 | docs/adrs/ADR-005-garantia-at-least-once-com-event-id.md | Trade-off | Responsabilidade de deduplicação transferida ao cliente | TRANSCRICAO | [09:25] Sofia |
| ADR-005-CONS-01 | docs/adrs/ADR-005-garantia-at-least-once-com-event-id.md | Consequência | Exigência documentada com destaque no portal do desenvolvedor | TRANSCRICAO | [09:26] Marcos |

### ADR-006: Reuso máximo dos padrões arquiteturais já existentes no projeto ([ADR-006-reuso-dos-padroes-existentes-do-projeto.md](./adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Reuso máximo: `AppError`, Pino, error middleware, módulos, Zod, códigos de erro | TRANSCRICAO | [09:30] Larissa |
| ADR-006-CTX-01 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Contexto | Módulos com controller, service, repository, routes e schemas | CODIGO | src/modules/products/product.repository.ts |
| ADR-006-CTX-02 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Contexto | Classe base `AppError` | CODIGO | src/shared/errors/app-error.ts |
| ADR-006-CTX-03 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Contexto | `errorMiddleware` trata `AppError`, `ZodError` e `PrismaClientKnownRequestError` | CODIGO | src/middlewares/error.middleware.ts |
| ADR-006-CTX-04 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Contexto | Códigos `INSUFFICIENT_STOCK` e `INVALID_STATUS_TRANSITION` como referência | CODIGO | src/shared/errors/http-errors.ts |
| ADR-006-DEC-02 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Novo módulo `src/modules/webhooks` com a estrutura padrão | TRANSCRICAO | [09:27] Bruno |
| ADR-006-DEC-03 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Lógica do worker em `webhook.worker.ts` ou `webhook.processor.ts` | TRANSCRICAO | [09:28] Bruno |
| ADR-006-DEC-04 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Prefixo `WEBHOOK_` nos códigos de erro | TRANSCRICAO | [09:29] Larissa |
| ADR-006-DEC-05 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Logger Pino reaproveitado, sem biblioteca nova | TRANSCRICAO | [09:29] Bruno |
| ADR-006-DEC-06 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Reuso de `requireRole` no replay da DLQ | TRANSCRICAO | [09:36] Larissa |
| ADR-006-DEC-07 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Convenção de PK UUID mantida | TRANSCRICAO | [09:51] Larissa |
| ADR-006-ALT-01 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Alternativa descartada | Convenções e bibliotecas novas só para webhooks | TRANSCRICAO | [09:29] Bruno |
| ADR-006-TO-01 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Trade-off | Consistência no lugar de estrutura sob medida, dado o prazo de 3 sprints | TRANSCRICAO | [09:46] Larissa |

### ADR-007: Ordenação de entrega garantida apenas por order_id, dependente de single-worker ([ADR-007-ordenacao-por-order-id-sem-garantia-global.md](./adrs/ADR-007-ordenacao-por-order-id-sem-garantia-global.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-007 | docs/adrs/ADR-007-ordenacao-por-order-id-sem-garantia-global.md | Restrição | Ordem garantida só por `order_id` e só com single-worker | TRANSCRICAO | [09:13] Larissa |
| ADR-007-CTX-01 | docs/adrs/ADR-007-ordenacao-por-order-id-sem-garantia-global.md | Contexto | Pergunta: `PAID` → `PROCESSING` → `SHIPPED` chegam em ordem? | TRANSCRICAO | [09:12] Larissa |
| ADR-007-CTX-02 | docs/adrs/ADR-007-ordenacao-por-order-id-sem-garantia-global.md | Contexto | Transições `PAID` → `PROCESSING` → `SHIPPED` → `DELIVERED` do pedido | CODIGO | src/modules/orders/order.status.ts |
| ADR-007-CTX-03 | docs/adrs/ADR-007-ordenacao-por-order-id-sem-garantia-global.md | Contexto | Single-worker processa em ordem de `created_at` | TRANSCRICAO | [09:12] Diego |
| ADR-007-ALT-01 | docs/adrs/ADR-007-ordenacao-por-order-id-sem-garantia-global.md | Alternativa descartada | Particionamento por `order_id` ou lock pessimista desde já | TRANSCRICAO | [09:13] Diego |
| ADR-007-TO-01 | docs/adrs/ADR-007-ordenacao-por-order-id-sem-garantia-global.md | Trade-off | Clientes não precisam de ordem global; aceita teto de escala | TRANSCRICAO | [09:14] Marcos |

### ADR-008: Payload do evento renderizado (snapshot) no momento da inserção na outbox ([ADR-008-snapshot-do-payload-na-insercao-do-outbox.md](./adrs/ADR-008-snapshot-do-payload-na-insercao-do-outbox.md))

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-008 | docs/adrs/ADR-008-snapshot-do-payload-na-insercao-do-outbox.md | Decisão | Payload renderizado como snapshot na inserção da outbox | TRANSCRICAO | [09:52] Bruno |
| ADR-008-CTX-01 | docs/adrs/ADR-008-snapshot-do-payload-na-insercao-do-outbox.md | Contexto | Dúvida: payload renderizado ou só `order_id` | TRANSCRICAO | [09:51] Bruno |
| ADR-008-ALT-01 | docs/adrs/ADR-008-snapshot-do-payload-na-insercao-do-outbox.md | Alternativa descartada | Renderizar sob demanda no envio | TRANSCRICAO | [09:52] Larissa |
| ADR-008-TO-01 | docs/adrs/ADR-008-snapshot-do-payload-na-insercao-do-outbox.md | Trade-off | Mais armazenamento por evento em troca de consistência temporal | TRANSCRICAO | [09:52] Diego |
| ADR-008-CONS-01 | docs/adrs/ADR-008-snapshot-do-payload-na-insercao-do-outbox.md | Consequência | Payload enxuto, sem `items`, limita o custo do snapshot | TRANSCRICAO | [09:43] Diego |

## Itens sem origem direta

Os itens abaixo aparecem nos documentos mas não foram discutidos na reunião nem decorrem diretamente do código. São definições de design feitas na escrita do FDD/PRD para fechar lacunas, e cada documento os sinaliza como tal. Ficam fora da tabela para não atribuir a eles uma origem que não têm; devem ser validados na sessão de revisão do design (`[09:50] Larissa`).

| Documento | Item |
| --- | --- |
| docs/FDD.md | Recuperação de linhas presas em `PROCESSING` há mais de 60 segundos |
| docs/FDD.md | Lote de 20 eventos por ciclo (`WEBHOOK_BATCH_SIZE`) e envio paralelo dentro do lote |
| docs/FDD.md | Uma linha de outbox por endpoint assinante |
| docs/FDD.md | Endpoint `GET /admin/webhooks/dead-letter` para listar a DLQ |
| docs/FDD.md | Duas assinaturas em `X-Signature` durante a carência de rotação |
| docs/FDD.md | Semântica de `WEBHOOK_SECRET_REQUIRED` (o código foi citado na reunião, o uso não) |
| docs/FDD.md | Corpo da resposta truncado em 2KB em `webhook_deliveries` |
| docs/FDD.md | Secret armazenada em texto recuperável; proteção em repouso delegada à revisão de segurança |
| docs/FDD.md | Risco de URL apontando para rede interna (SSRF), levado à revisão de segurança |
| docs/FDD.md | Métricas derivadas de logs (`webhook_worker_stats`) e alertas sugeridos |
| docs/PRD.md | R8: processo de entregas parado atrasa as notificações de todos os clientes |
