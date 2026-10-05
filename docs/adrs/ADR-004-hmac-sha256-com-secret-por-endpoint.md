# ADR-004: Autenticação dos webhooks via HMAC-SHA256 com secret por endpoint

## Status

Aceito

## Data

Decisão tomada em reunião técnica de kickoff da feature (ver `TRANSCRICAO.md`). ADR registrado em 2026-10-05.

## Decisores

Sofia (Eng. de Segurança), Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)

## Contexto

Como os eventos de webhook carregam dados de pedidos para fora da infraestrutura da empresa (`[09:19] Sofia`), o cliente que recebe a chamada precisa conseguir validar que (a) a requisição realmente partiu da plataforma e (b) o payload não foi adulterado em trânsito.

`[09:20] Sofia` propôs o padrão HMAC: assinar o corpo da requisição com uma secret compartilhada entre a plataforma e o cliente, enviando a assinatura em um header (`X-Signature`); o cliente verifica a assinatura do lado dele. `[09:20] Bruno` perguntou qual algoritmo usar; `[09:20] Sofia` definiu HMAC-SHA256 como padrão de mercado, amplamente suportado por bibliotecas de terceiros.

`[09:21] Sofia` acrescentou um requisito crítico: **cada endpoint de webhook de cliente precisa ter sua própria secret**, não uma secret global da plataforma — caso contrário, o vazamento de uma secret comprometeria a integridade de todos os clientes. Isso implica que a tabela de configuração de webhook armazena `url`, `secret`, `customer_id` e estado ativo (`[09:21] Bruno`, confirmado por Sofia).

`[09:21] Sofia` também exigiu que a secret seja **rotacionável** via API, com **grace period de 24 horas**: ao rotacionar, a secret antiga continua válida em paralelo por 24h, dando tempo ao cliente de migrar seus sistemas antes de a secret anterior ser definitivamente invalidada. `[09:22] Diego` justificou essa exigência citando um incidente real: um cliente já vazou uma secret em log de aplicação no passado.

## Decisão

Implementar autenticação de webhooks via **HMAC-SHA256** calculado sobre o corpo (body) da requisição, enviado no header `X-Signature`. Cada endpoint de webhook cadastrado por um cliente possui **uma secret própria e exclusiva**, gerada pela plataforma na criação do cadastro (não definida pelo cliente) e armazenada junto a `url`, `customer_id` e estado ativo do webhook.

A plataforma expõe um mecanismo de **rotação de secret** via API. Durante a rotação, a secret anterior permanece válida por um **grace period de 24 horas** em paralelo à nova, após o qual é definitivamente invalidada.

## Alternativas Consideradas

### Secret única/global por plataforma

Usar uma única secret compartilhada entre a plataforma e todos os clientes para assinatura HMAC.

- Descartada porque o vazamento de uma única secret comprometeria a autenticidade de **todos** os endpoints de webhook de **todos** os clientes simultaneamente, ao invés de um incidente isolado e contido a um único cliente (`[09:21] Sofia`).

### Rotação imediata, sem grace period

Invalidar a secret antiga no exato momento em que uma nova é emitida.

- Descartada (implicitamente, ao decidir pelo grace period) porque obrigaria o cliente a atualizar a secret do lado dele de forma perfeitamente sincronizada com a plataforma, sob risco de todas as entregas falharem por assinatura inválida durante a janela de migração. O grace period de 24h existe justamente para evitar essa janela de falha coordenada (`[09:21] Sofia`).

## Consequências

### Positivas

- Isola o raio de impacto de um vazamento de secret a um único endpoint/cliente, em vez de comprometer toda a base de clientes.
- Permite que o próprio cliente rotacione sua secret proativamente (por exemplo, após suspeitar de vazamento em seus próprios logs — cenário já observado, `[09:22] Diego`), sem depender de intervenção da equipe da plataforma.
- HMAC-SHA256 é um padrão amplamente adotado no mercado (citado por Sofia como equivalente ao usado por Stripe e GitHub em `[09:25]`), reduzindo a barreira de integração para os clientes.

### Negativas

- Aumenta a superfície de dados sensíveis a proteger: cada registro de configuração de webhook agora guarda uma secret por cliente, exigindo cuidado de armazenamento e de não vazamento em logs (reforçado pelo incidente citado por Diego).
- O grace period de 24h mantém duas secrets simultaneamente válidas por endpoint durante a janela de rotação, uma janela em que uma secret potencialmente comprometida ainda é aceita.
- Exige validação adicional no schema de cadastro do webhook (por exemplo, obrigatoriedade de URL em HTTPS, discutida em `[09:23] Sofia`) e lógica de verificação de múltiplas secrets válidas simultaneamente durante a rotação.

### Trade-off explícito

Aceitamos a complexidade operacional de manter potencialmente duas secrets válidas por endpoint durante 24h em troca de uma migração segura e sem interrupção de entregas para o cliente durante a rotação — a alternativa (corte imediato) transferiria o risco de indisponibilidade para o cliente no pior momento possível (logo após uma suspeita de vazamento).
