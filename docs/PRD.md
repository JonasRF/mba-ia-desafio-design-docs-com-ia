# PRD — Sistema de Webhooks de Notificação de Pedidos

## Metadados

| Campo | Valor |
| --- | --- |
| **Autor** | Marcos (Product Manager) |
| **Status** | Em revisão |
| **Data** | 2026-10-08 |
| **Revisores** | Larissa (Tech Lead), Bruno (Eng. Pleno, Pedidos), Diego (Eng. Sênior, Plataforma), Sofia (Eng. de Segurança) |
| **Origem** | Reunião técnica de kickoff (`TRANSCRICAO.md`) |
| **Documentos relacionados** | [RFC](./RFC.md), [FDD](./FDD.md), [ADRs](./adrs/), [Tracker](./TRACKER.md) |

Este documento responde *por que* e *o quê*. A arquitetura proposta está no [RFC](./RFC.md), cada decisão isolada nos [ADRs](./adrs/) e a especificação de implementação no [FDD](./FDD.md). As referências no formato `[hh:mm] Nome` apontam para a fala de origem na transcrição.

## 1. Resumo e contexto da feature

O Order Management System passará a **avisar os clientes B2B, por webhook, sempre que o status de um pedido deles mudar**. O cliente cadastra uma URL `https`, escolhe quais status quer acompanhar e passa a receber uma requisição assinada a cada transição, em menos de 10 segundos.

Hoje a plataforma não tem nenhum mecanismo de notificação externa: a única forma de um cliente saber que um pedido mudou é consultar `GET /orders` repetidamente. Três clientes pediram formalmente a mudança na semana anterior à reunião (`[09:00] Marcos`).

O fluxo é somente de saída, da plataforma para o cliente (`[09:02] Marcos`). A entrega é feita apenas por API; não há interface visual nesta fase (`[09:40] Larissa`).

## 2. Problema e motivação

**Problema.** Atlas Comercial, MaxDistribuição e Nova Cargo descobrem mudanças de status consultando `GET /orders` de tempos em tempos. Isso deixa a integração deles **lenta** (a informação só chega na próxima consulta) e **cara** (a maior parte das consultas não traz novidade) (`[09:00] Marcos`).

**Motivação de negócio.** A Atlas sinalizou que pode migrar para um concorrente se a entrega não ocorrer até o fim do trimestre, e espera a feature para o fim de novembro (`[09:00] Marcos`, `[09:45] Marcos`). O risco imediato é perda de receita de um cliente; os outros dois têm o mesmo pedido em aberto.

**O que o cliente entende por "tempo real".** Qualquer coisa abaixo de 10 segundos. O que importa é a informação não ficar pendurada nem exigir atualização manual (`[09:02] Marcos`).

**Restrição que molda a solução.** A mudança de status é uma operação crítica e já pesada. Ela não pode ficar mais lenta nem falhar porque o sistema de um cliente está lento ou fora do ar (`[09:04] Bruno`).

## 3. Público-alvo e cenários de uso

| Persona | Quem é | O que precisa |
| --- | --- | --- |
| **Cliente B2B integrador** | Equipe técnica de clientes como Atlas Comercial, MaxDistribuição e Nova Cargo, que opera um sistema próprio integrado à plataforma | Saber que o pedido mudou sem ficar consultando; confiar que a notificação veio da plataforma |
| **Usuário que representa o cliente** | Usuário autenticado no OMS que gerencia a integração em nome do cliente (`[09:32] Marcos`) | Cadastrar, ajustar e diagnosticar os webhooks pela API |
| **Administrador da plataforma** | Usuário com role `ADMIN` | Reprocessar notificações que esgotaram as tentativas |
| **Product Manager** | Marcos | Documentar a integração no portal do desenvolvedor e confirmar o prazo com os clientes |

**Cenários de uso**

1. **Acompanhar só o que interessa.** Um cliente cadastra um webhook pedindo apenas `SHIPPED` e `DELIVERED`. Mudanças para outros status não geram notificação para ele (`[09:33] Marcos`).
2. **Confiar na origem.** Ao receber a notificação, o sistema do cliente confere a assinatura com a secret que recebeu no cadastro e descarta requisições que não conferem (`[09:19] Sofia`).
3. **Atravessar uma indisponibilidade.** O cliente faz uma manutenção planejada de duas horas. As notificações do período são retentadas e chegam depois que ele volta, sem intervenção de ninguém (`[09:16] Diego`).
4. **Receber a mesma notificação duas vezes.** Uma entrega é repetida. O cliente reconhece o identificador do evento e ignora a duplicata (`[09:25] Diego`).
5. **Trocar uma secret exposta.** A secret do cliente vaza em um log da aplicação dele. Ele pede uma nova pela API e tem 24 horas para atualizar os sistemas, sem perder notificações no intervalo (`[09:21] Sofia`, `[09:22] Diego`).
6. **Investigar um problema.** O cliente consulta as últimas entregas do webhook dele, com sucesso ou falha, payload, resposta e tempo de resposta (`[09:34] Marcos`).
7. **Recuperar o que falhou de vez.** O cliente ficou fora do ar por mais tempo do que as retentativas cobrem. Um administrador reprocessa as notificações e a ação fica registrada com o autor (`[09:18] Diego`, `[09:36] Sofia`).

## 4. Objetivos e métricas de sucesso

| # | Objetivo | Métrica | Meta | Origem |
| --- | --- | --- | --- | --- |
| O1 | Notificar o cliente em "tempo real" | Tempo entre a mudança de status e a primeira tentativa de entrega | **Menos de 10 segundos** | `[09:02] Marcos` |
| O2 | Atender os clientes que pediram a feature, dentro do prazo | Feature em produção, com revisão de segurança concluída | **Até o fim de novembro, em 3 sprints** | `[09:45] Marcos`, `[09:46] Larissa` |
| O3 | Substituir o polling como forma de acompanhar pedidos | Clientes solicitantes com webhook ativo | **3 de 3** (Atlas Comercial, MaxDistribuição, Nova Cargo) | `[09:00] Marcos` |
| O4 | Não perder notificação quando o cliente fica indisponível | Janela de indisponibilidade do cliente coberta por retentativas automáticas | **Cerca de 15 horas** (5 tentativas) | `[09:17] Diego` |
| O5 | Não degradar a operação de pedidos | Mudanças de status que commitam sem evento registrado | **Zero** | `[09:40] Bruno` |

**Como medir.** O1 é acompanhado pela idade do evento pendente mais antigo e O4 pelo volume de eventos em dead letter, ambos descritos na seção de observabilidade do [FDD](./FDD.md#9-observabilidade). O5 é garantido por construção e verificado em teste (seção 12).

**O que ainda não tem meta.** A reunião não definiu linha de base para o volume atual de consultas a `GET /orders` nem meta de taxa de sucesso de entrega. As duas ficam para depois da primeira medição em produção; a decisão sobre aviso por email e rate limiting depende dessa medição (`[09:37] Larissa`, `[09:39] Diego`).

## 5. Escopo

### Incluso

- Cadastro, edição, remoção e listagem de webhooks por cliente, via API.
- Escolha, por webhook, dos status de pedido que geram notificação.
- Notificação a cada mudança de status de pedido, com payload enxuto e assinado.
- Rotação de secret pela API, com 24 horas de carência.
- Retentativas automáticas e isolamento das falhas permanentes em dead letter.
- Reprocessamento manual de dead letter por administrador, com auditoria.
- Consulta ao histórico de entregas de cada webhook.
- Documentação da integração no portal do desenvolvedor (`[09:40] Marcos`).

### Fora de escopo

| Item | Situação | Motivo | Origem |
| --- | --- | --- | --- |
| **Aviso por email** quando o webhook do cliente falha repetidamente | Adiado para a próxima fase | Decidir depois de medir o impacto | `[09:37] Marcos`, `[09:37] Larissa` |
| **Dashboard visual** para o cliente ver os webhooks dele | Descartado nesta feature | Projeto separado, do time de frontend; aqui só endpoints | `[09:39] Marcos`, `[09:40] Larissa` |
| **Rate limiting de envio** para o cliente | Adiado: observar e decidir depois | Só implementar se virar problema | `[09:38] Diego`, `[09:39] Larissa` |
| **Webhooks de entrada** (cliente enviando para a plataforma) | Descartado | Os clientes querem receber, não enviar | `[09:02] Marcos` |
| **Garantia de entrega exatamente uma vez** | Descartado | Exigiria coordenação dos dois lados; complexidade desproporcional | `[09:25] Diego` |
| **Garantia de ordem global** entre pedidos | Descartado | Os clientes nunca pediram; a ordem vale por pedido | `[09:13] Larissa`, `[09:14] Marcos` |
| **Múltiplos workers em paralelo** | Adiado | "Problema do futuro"; hoje um único worker atende | `[09:13] Diego` |
| **Arquivamento** de eventos já entregues | Adiado | Fora do escopo desta feature | `[09:08] Diego` |
| **Permissões mais restritas** no gerenciamento de webhooks | Adiado | Qualquer role autenticada por enquanto; endurecer mais à frente | `[09:37] Sofia` |

## 6. Requisitos funcionais

| ID | Requisito | Origem |
| --- | --- | --- |
| RF-01 | O usuário autenticado cadastra um webhook informando o cliente, a URL de destino e a lista de status que quer receber. O cliente é informado na requisição; não é deduzido do JWT. | `[09:31] Marcos`, `[09:32] Larissa` |
| RF-02 | A secret do webhook é gerada pela plataforma e devolvida na resposta do cadastro. | `[09:31] Marcos` |
| RF-03 | O usuário edita um webhook existente. | `[09:33] Bruno` |
| RF-04 | O usuário remove um webhook. | `[09:33] Bruno` |
| RF-05 | O usuário lista os webhooks de um cliente. | `[09:33] Bruno` |
| RF-06 | Cada webhook define quais status de pedido quer ouvir. Uma mudança para um status que nenhum webhook do cliente assina não gera evento. | `[09:33] Marcos`, `[09:34] Bruno` |
| RF-07 | Toda mudança de status de pedido para um status assinado gera um evento, registrado junto com a própria mudança: ou as duas coisas acontecem, ou nenhuma. | `[09:40] Bruno`, `[09:41] Diego` |
| RF-08 | A notificação é um JSON com `event_id`, `event_type` (`order.status_changed`), `timestamp` em ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e `total_cents`. Os itens do pedido não são enviados. | `[09:43] Diego` |
| RF-09 | O conteúdo da notificação reflete o pedido no momento da mudança de status, mesmo que a entrega aconteça depois. | `[09:52] Larissa`, `[09:52] Diego` |
| RF-10 | Toda notificação leva os headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json`. | `[09:44] Diego`, `[09:44] Sofia` |
| RF-11 | Toda notificação é assinada com HMAC-SHA256 sobre o corpo, usando a secret do webhook de destino. | `[09:20] Sofia`, `[09:22] Sofia` |
| RF-12 | O usuário solicita uma nova secret pela API. A anterior continua válida por 24 horas e depois deixa de valer. | `[09:21] Sofia` |
| RF-13 | Uma entrega que falha é retentada automaticamente até 5 vezes, com intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. | `[09:15] Diego`, `[09:17] Larissa` |
| RF-14 | Esgotadas as tentativas, o evento é movido para uma dead letter, com payload, motivo da falha e data. | `[09:18] Diego` |
| RF-15 | Um administrador reprocessa manualmente um evento da dead letter, que volta a ser entregue. | `[09:18] Diego`, `[09:35] Diego` |
| RF-16 | O usuário consulta o histórico de entregas de um webhook (as últimas 100), com sucesso ou falha, payload, resposta e tempo de resposta. | `[09:34] Marcos` |

Os contratos dos endpoints (rotas, exemplos e códigos de erro) estão na seção 6 do [FDD](./FDD.md#6-contratos-públicos).

## 7. Requisitos não funcionais

| ID | Categoria | Requisito | Origem |
| --- | --- | --- | --- |
| RNF-01 | Latência | A notificação parte em menos de 10 segundos após a mudança de status. A verificação de eventos pendentes ocorre a cada 2 segundos. | `[09:02] Marcos`, `[09:10] Larissa` |
| RNF-02 | Confiabilidade | Entrega at-least-once: nenhum evento registrado é perdido, e duplicatas são possíveis. | `[09:24] Diego`, `[09:26] Larissa` |
| RNF-03 | Isolamento | A lentidão ou indisponibilidade de um cliente não atrasa nem impede mudanças de status. | `[09:04] Bruno` |
| RNF-04 | Disponibilidade | As entregas rodam em um processo separado da API; reiniciar a API não interrompe as entregas. | `[09:11] Diego` |
| RNF-05 | Ordenação | Eventos de um mesmo pedido chegam na ordem em que as mudanças ocorreram. Não há garantia de ordem entre pedidos diferentes. | `[09:12] Diego`, `[09:13] Larissa` |
| RNF-06 | Tempo de resposta | O cliente tem 10 segundos para responder; depois disso a entrega conta como falha e é retentada. | `[09:42] Diego` |
| RNF-07 | Segurança | Só são aceitas URLs `https`. Cadastro com `http` é recusado com erro de validação. | `[09:23] Sofia` |
| RNF-08 | Segurança | Cada webhook tem secret própria. Não existe secret global da plataforma. | `[09:21] Sofia` |
| RNF-09 | Segurança | O reprocessamento de dead letter exige role `ADMIN`. Os demais endpoints exigem apenas autenticação. | `[09:36] Sofia`, `[09:36] Larissa`, `[09:37] Sofia` |
| RNF-10 | Auditoria | Todo reprocessamento registra quem o executou. | `[09:36] Sofia` |
| RNF-11 | Limite | Payload acima de 64KB gera erro; não é truncado nem enviado. | `[09:23] Sofia`, `[09:24] Larissa` |
| RNF-12 | Operação | Nenhuma infraestrutura nova: a feature usa o MySQL e a stack já existentes. | `[09:07] Diego`, `[09:11] Diego` |
| RNF-13 | Manutenibilidade | A feature segue os padrões do projeto: módulo em `src/modules`, `AppError`, logger Pino, middleware de erro, schemas Zod e códigos de erro com prefixo `WEBHOOK_`. | `[09:27] Bruno`, `[09:29] Larissa`, `[09:30] Larissa` |

## 8. Decisões e trade-offs principais

O que cada decisão significa para o produto. A justificativa técnica completa está no ADR indicado.

| Decisão | O que ganhamos | O que aceitamos em troca | ADR |
| --- | --- | --- | --- |
| Registrar o evento junto com a mudança de status e entregar depois | A mudança de status não depende do cliente; nenhum evento se perde | A notificação não é instantânea | [ADR-001](./adrs/ADR-001-outbox-no-mysql.md) |
| Verificar eventos pendentes a cada 2 segundos, em processo separado | Atende ao limite de 10 segundos sem infraestrutura nova | Até 2 segundos de espera no pior caso; um processo a mais para operar | [ADR-002](./adrs/ADR-002-worker-dedicado-em-polling.md) |
| 5 tentativas em cerca de 15 horas, depois dead letter | Cobre manutenções planejadas de horas; nada fica pendurado para sempre | Depois de ~15 horas a recuperação depende de ação manual de um administrador | [ADR-003](./adrs/ADR-003-retry-com-backoff-e-dead-letter-queue.md) |
| Assinatura HMAC-SHA256 com secret por webhook, rotacionável | Padrão de mercado; o vazamento de uma secret afeta um único webhook | O cliente precisa implementar a verificação e guardar a secret | [ADR-004](./adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md) |
| Entrega at-least-once com `X-Event-Id` | Simplicidade; mesmo modelo de Stripe e GitHub | A deduplicação fica por conta do cliente | [ADR-005](./adrs/ADR-005-garantia-at-least-once-com-event-id.md) |
| Reuso máximo dos padrões do projeto | Prazo de 3 sprints viável; nada novo para o time aprender | Sem ferramentas novas nesta fase | [ADR-006](./adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) |
| Ordem garantida só por pedido | Atende ao que os clientes pediram | Escalar as entregas no futuro exigirá nova decisão | [ADR-007](./adrs/ADR-007-ordenacao-por-order-id-sem-garantia-global.md) |
| Notificação com o retrato do pedido no momento da mudança | Uma entrega tardia não mostra um estado posterior | Para dados atuais ou itens, o cliente consulta `GET /orders/:id` | [ADR-008](./adrs/ADR-008-snapshot-do-payload-na-insercao-do-outbox.md) |
| Payload enxuto, sem itens do pedido | Notificações pequenas, longe do limite de 64KB | Uma consulta extra quando o cliente quer detalhes | `[09:43] Diego` |

## 9. Dependências

| Dependência | Tipo | Responsável | Origem |
| --- | --- | --- | --- |
| Revisão de segurança do HMAC e da geração de secret, com pelo menos 2 dias úteis antes do deploy | Processo | Sofia | `[09:46] Sofia`, `[09:49] Sofia` |
| Documentação no portal do desenvolvedor: como integrar via API, validar a assinatura e deduplicar por `X-Event-Id` | Entrega paralela | Marcos | `[09:26] Marcos`, `[09:40] Marcos` |
| Cliente expõe um endpoint `https`, valida a assinatura, deduplica e responde em até 10 segundos | Externa | Clientes B2B | `[09:23] Sofia`, `[09:24] Diego`, `[09:42] Diego` |
| Usuários que representam o cliente, autenticados com o JWT do sistema | Pré-requisito existente | Plataforma | `[09:32] Marcos` |
| Módulo de pedidos: a mudança de status em `src/modules/orders/order.service.ts` passa a registrar o evento | Código existente | Bruno | `[09:40] Bruno` |
| Controle de role já existente (`requireRole`, em `src/middlewares/auth.middleware.ts`) | Código existente | — | `[09:36] Larissa` |
| Banco MySQL já em uso, compartilhado entre a API e o processo de entregas | Infraestrutura existente | Plataforma | `[09:07] Diego`, `[09:11] Bruno` |
| Sessão de revisão do design com Bruno e Diego antes do início da implementação | Processo | Larissa | `[09:50] Larissa` |
| Confirmação do prazo com a Atlas | Comunicação | Marcos | `[09:47] Marcos` |

## 10. Riscos e mitigação

Probabilidade e impacto são estimativas, abertas à revisão.

| # | Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- | --- |
| R1 | O prazo de 3 sprints estoura e a Atlas migra para o concorrente | Média | Alto | Reuso dos padrões existentes; email, dashboard e rate limiting fora do escopo; revisão de segurança agendada com antecedência (`[09:46] Larissa`, `[09:49] Sofia`) |
| R2 | O cliente não deduplica e processa a mesma notificação duas vezes | Média | Médio | `X-Event-Id` igual em todas as tentativas; destaque na documentação do portal (`[09:26] Marcos`) |
| R3 | O webhook do cliente falha e ele não percebe, porque não há aviso por email nesta fase | Média | Médio | Histórico de entregas consultável pelo cliente; reprocessamento por administrador; aviso por email reavaliado na próxima fase (`[09:37] Larissa`) |
| R4 | O cliente fica fora do ar por mais de ~15 horas e os eventos vão para a dead letter | Baixa | Médio | Reprocessamento manual auditado; "se um cliente cair por 15 horas, ele já está com problema sério dele" (`[09:17] Marcos`) |
| R5 | A secret vaza do lado do cliente (já aconteceu em log de aplicação) | Média | Médio | Secret por webhook limita o alcance; rotação pela API com 24 horas de carência (`[09:21] Sofia`, `[09:22] Diego`) |
| R6 | Uma rajada de mudanças de status sobrecarrega o endpoint do cliente (50 pedidos em um minuto geram 50 chamadas) | Baixa | Médio | Sem mitigação nesta fase; observar o volume por webhook e decidir (`[09:38] Diego`) |
| R7 | Um problema no registro do evento impede mudanças de status | Baixa | Alto | Comportamento intencional, para nunca haver status alterado sem evento; coberto por testes de integração e pela suíte existente de pedidos (`[09:40] Bruno`) |
| R8 | O processo de entregas para e as notificações atrasam para todos os clientes | Média | Alto | Os eventos ficam registrados e saem na retomada; alerta sobre a idade do evento pendente mais antigo ([FDD](./FDD.md#9-observabilidade)) |

Os riscos de implementação estão na seção 13 do [FDD](./FDD.md#13-riscos-e-mitigação).

## 11. Critérios de aceitação

A feature é aceita quando todos os itens abaixo forem demonstrados.

1. Um usuário autenticado cadastra um webhook com URL `https` e uma lista de status, e recebe a secret na resposta. *(RF-01, RF-02)*
2. O cadastro com URL `http` é recusado com erro de validação. *(RNF-07)*
3. O usuário edita, remove e lista os webhooks de um cliente. *(RF-03, RF-04, RF-05)*
4. Uma mudança para um status assinado gera notificação; uma mudança para um status não assinado não gera. *(RF-06, RF-07)*
5. A notificação parte em menos de 10 segundos após a mudança de status. *(RNF-01)*
6. A notificação traz os campos de RF-08, sem itens do pedido, e os headers de RF-10. *(RF-08, RF-10)*
7. A assinatura recebida confere com o HMAC-SHA256 do corpo, calculado com a secret do webhook. *(RF-11)*
8. Após a rotação, a notificação é validável com a secret nova e com a anterior por 24 horas; depois, só com a nova. *(RF-12)*
9. Com o cliente indisponível, a entrega é retentada nos intervalos de RF-13 e, esgotadas as tentativas, o evento aparece na dead letter. *(RF-13, RF-14)*
10. Todas as tentativas de um mesmo evento levam o mesmo `X-Event-Id`. *(RNF-02)*
11. Mudanças sucessivas de um mesmo pedido chegam ao cliente na ordem em que ocorreram. *(RNF-05)*
12. Um usuário sem role `ADMIN` não consegue reprocessar a dead letter; um `ADMIN` consegue, e o registro identifica quem executou. *(RF-15, RNF-09, RNF-10)*
13. O histórico de entregas mostra sucesso ou falha, payload, resposta e tempo de resposta. *(RF-16)*
14. Com o cliente fora do ar, as mudanças de status continuam funcionando normalmente. *(RNF-03)*
15. Se o evento não puder ser registrado, o status do pedido não muda. *(RF-07)*
16. A revisão de segurança de Sofia foi concluída antes do deploy.
17. A documentação de integração está publicada no portal do desenvolvedor.

Os critérios de aceite técnicos, verificáveis em código, estão na seção 12 do [FDD](./FDD.md#12-critérios-de-aceite-técnicos).

## 12. Estratégia de testes e validação

**Testes automatizados.** Seguem o que o projeto já usa: Vitest e Supertest, com banco real, no molde de `tests/orders.test.ts`. O endpoint do cliente é simulado, de modo que os testes exercitam sucesso, erro, timeout e indisponibilidade sem depender de rede.

| Camada | O que valida | Critérios |
| --- | --- | --- |
| API de configuração | Cadastro, edição, remoção, listagem, rotação de secret, recusa de `http`, permissões | 1, 2, 3, 12 |
| Integração com pedidos | Evento registrado junto com a mudança de status; filtro por status; rollback quando o registro falha | 4, 15 |
| Entrega | Tempo até a primeira tentativa, payload, headers, assinatura, identificador estável, ordem por pedido | 5, 6, 7, 8, 10, 11 |
| Falha e recuperação | Retentativas, dead letter, reprocessamento, histórico de entregas, mudança de status com o cliente fora do ar | 9, 12, 13, 14 |
| Regressão | `tests/orders.test.ts` e `tests/auth.test.ts` continuam passando sem alteração | — |

Os testes ponta a ponta estão previstos na estimativa, junto com a integração no módulo de pedidos (`[09:46] Larissa`).

**Revisão de segurança.** Sofia revisa o HMAC e a geração de secret, com pelo menos dois dias úteis reservados antes do deploy (`[09:46] Sofia`). É condição para subir (critério 16).

**Validação em produção.** Depois do deploy, acompanhar o tempo até a primeira tentativa (O1), o volume de eventos em dead letter (O4) e o volume de notificações por webhook, que é o sinal para a decisão adiada sobre rate limiting (`[09:39] Diego`).

**Validação com os clientes.** A meta O3 é verificada pelo cadastro e uso de webhooks por Atlas Comercial, MaxDistribuição e Nova Cargo, apoiados pela documentação do portal (`[09:40] Marcos`).
