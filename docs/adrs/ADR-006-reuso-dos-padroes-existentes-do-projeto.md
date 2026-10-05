# ADR-006: Reuso máximo dos padrões arquiteturais já existentes no projeto

## Status

Aceito

## Data

Decisão tomada em reunião técnica de kickoff da feature (ver `TRANSCRICAO.md`). ADR registrado em 2026-10-05.

## Decisores

Bruno (Eng. Pleno, Pedidos), Diego (Eng. Sênior, Plataforma), Larissa (Tech Lead)

## Contexto

A partir de `[09:27] Larissa` a discussão passou de decisões de arquitetura de dados/entrega para a estrutura de código da feature. `[09:27] Bruno` observou que o projeto já tem um padrão claro: cada domínio é um módulo em `src/modules/<dominio>` com `controller`, `service`, `repository`, `routes` e `schemas` — visível hoje em `src/modules/orders/`, `src/modules/customers/`, `src/modules/products/` e `src/modules/users/` (cada um com os cinco arquivos nesse padrão, ex.: `order.controller.ts`, `order.service.ts`, `order.repository.ts`, `order.routes.ts`, `order.schemas.ts`). A proposta de Bruno, aceita por Diego, foi criar `src/modules/webhooks/` seguindo exatamente essa estrutura.

`[09:28] Bruno` também localizou onde entra o processamento assíncrono: `src/worker.ts` como novo entry-point (paralelo ao `src/server.ts` existente), com a lógica de processamento residindo em um arquivo dentro do próprio módulo (`src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts`).

Sobre tratamento de erros, `[09:28] Bruno` apontou o padrão já em uso: a classe base `AppError` (`src/shared/errors/app-error.ts`) e subclasses específicas como `InsufficientStockError` e `InvalidStatusTransitionError` (`src/shared/errors/http-errors.ts`), cada uma com um código de erro em `SCREAMING_SNAKE_CASE` (ex.: `INSUFFICIENT_STOCK`, `INVALID_STATUS_TRANSITION`). A decisão foi replicar esse padrão para o módulo de webhooks com um prefixo próprio, `WEBHOOK_` (ex.: `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`), confirmado por `[09:29] Larissa`.

`[09:29] Bruno` destacou ainda que o logger (Pino, configurado em `src/shared/logger/index.ts`) e o middleware de erro centralizado (`src/middlewares/error.middleware.ts`, que já trata instâncias de `AppError`, `ZodError` e `Prisma.PrismaClientKnownRequestError`) não precisam de nenhuma alteração: como os erros do módulo de webhooks estenderão `AppError`, o middleware existente já os trata automaticamente.

`[09:29]-[09:30] Diego/Bruno` fecharam o ponto de infraestrutura compartilhada: o `PrismaClient` é por processo, então o worker abre sua própria instância, mas conectada à mesma `DATABASE_URL` e ao mesmo schema Prisma (`prisma/schema.prisma`) já usado pela API — sem introduzir um client ou ORM diferente.

`[09:30] Larissa` consolidou a decisão: reuso máximo do que já existe — `AppError`, Pino, error middleware, padrão de módulos, padrão de schemas Zod, padrão de códigos de erro.

## Decisão

O módulo de webhooks seguirá, sem exceções, os padrões arquiteturais já estabelecidos no restante da base de código:

- **Estrutura modular**: `src/modules/webhooks/` com `webhook.controller.ts`, `webhook.service.ts`, `webhook.repository.ts`, `webhook.routes.ts` e `webhook.schemas.ts`, no mesmo molde dos módulos existentes (`src/modules/orders/`, `src/modules/customers/`, `src/modules/products/`, `src/modules/users/`).
- **Processamento assíncrono**: novo entry-point `src/worker.ts` (paralelo a `src/server.ts`), delegando a lógica de negócio para um arquivo dentro do módulo (`src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts`).
- **Erros de domínio**: subclasses de `AppError` (`src/shared/errors/app-error.ts`), seguindo o mesmo padrão de `src/shared/errors/http-errors.ts`, com códigos de erro prefixados por `WEBHOOK_`.
- **Logging**: reuso do logger Pino já configurado (`src/shared/logger/index.ts`), sem introduzir biblioteca de logging adicional.
- **Tratamento centralizado de erros HTTP**: reuso do `errorMiddleware` existente (`src/middlewares/error.middleware.ts`), sem necessidade de alteração, já que ele trata qualquer instância de `AppError`.
- **Autorização**: reuso do middleware `requireRole` já existente (`src/middlewares/auth.middleware.ts`) para proteger o endpoint administrativo de replay da DLQ.
- **Persistência**: mesmo banco MySQL e mesmo schema Prisma (`prisma/schema.prisma`), com convenção de chaves primárias UUID já em uso em todos os modelos existentes (`@id @default(uuid()) @db.Char(36)`).

## Alternativas Consideradas

### Introduzir convenções e bibliotecas novas específicas para o módulo de webhooks

Por exemplo, adotar uma biblioteca de filas/jobs em memória, um logger diferente para o worker, ou uma estrutura de pastas própria (ex.: arquitetura hexagonal isolada) só para este módulo, por ser um domínio "diferente" (integração externa) dos demais módulos de negócio.

- Descartada porque introduziria inconsistência arquitetural sem benefício claro: o time já tem convenções funcionando (estrutura de módulos, tratamento de erros, logging) e qualquer novo padrão aumentaria a curva de aprendizado e o custo de manutenção sem resolver um problema real (`[09:30] Larissa`: "reuso máximo do que já existe").

## Consequências

### Positivas

- Qualquer desenvolvedor já familiarizado com `src/modules/orders` ou `src/modules/customers` consegue navegar o módulo de webhooks sem curva de aprendizado adicional.
- O middleware de erro centralizado (`src/middlewares/error.middleware.ts`) funciona automaticamente para os novos erros, sem necessidade de alteração ou teste adicional nesse componente.
- Reduz a superfície de revisão de segurança e de código: Sofia revisa HMAC e geração de secret (`[09:46] Sofia`), não um novo framework de erros ou logging.

### Negativas

- Caso algum padrão existente tenha uma limitação (por exemplo, o `errorMiddleware` não distingue hierarquias de erro além de `instanceof AppError`), essa limitação é herdada pelo módulo de webhooks em vez de ser resolvida sob medida.
- Acopla a evolução do módulo de webhooks à evolução dos padrões compartilhados (`AppError`, error middleware, etc.): uma mudança de contrato nesses pontos compartilhados impacta também o módulo novo.

### Trade-off explícito

Abrimos mão da liberdade de desenhar uma estrutura "ideal" especificamente para o domínio de webhooks em troca de consistência com o restante da base de código e de menor custo de revisão e manutenção — importante dado o prazo apertado de três sprints (`[09:46] Larissa`).
