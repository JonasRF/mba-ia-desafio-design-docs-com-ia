# Da Reunião ao Documento: Design Docs Gerados por IA

Documentação do processo de produção do pacote de design docs da feature **Sistema de Webhooks de Notificação de Pedidos**. O enunciado original do desafio está preservado no histórico do repositório: [enunciado original](https://github.com/JonasRF/mba-ia-desafio-design-docs-com-ia/blob/e7f6311b96ee3b21bf8cb7077f53a6e4045143eb/README.md).

## Sobre o desafio

O ponto de partida era a transcrição de uma reunião técnica de cerca de 55 minutos (`TRANSCRICAO.md`), em que tech lead, PM, dois engenheiros e a engenheira de segurança decidem como um Order Management System em produção vai notificar clientes B2B, por webhook, sempre que o status de um pedido mudar. Fora a transcrição, só havia o código da aplicação (Node.js, TypeScript, Prisma e MySQL), que não tem nenhum mecanismo de eventos, filas ou notificação externa.

A tarefa foi transformar esse material em um pacote de documentação acionável (PRD, RFC, FDD, de 5 a 8 ADRs e um tracker de rastreabilidade), usando IA como ferramenta principal de produção. A regra central era que tudo nos documentos precisava ter origem identificável na transcrição ou no código: nada de requisitos inventados e, principalmente, nada do que a reunião descartou ou adiou registrado como requisito.

## Ferramentas de IA utilizadas

- **Claude Code (extensão do VS Code):** ambiente de todo o trabalho. O agente leu o código e a transcrição diretamente do repositório, escreveu e editou os arquivos em `docs/` e fez commits e push sob comando.
- **Claude Sonnet 5:** modelo da primeira sessão, que fez a exploração inicial do código e produziu os 8 ADRs.
- **Claude Opus 5.5:** modelo das sessões seguintes, que produziu o RFC, o FDD, o PRD e o tracker, as revisões dos documentos e a verificação cruzada contra a transcrição e o código.

## Workflow adotado

Cada documento foi produzido em uma sessão própria, com um prompt que trazia o papel do documento, as seções obrigatórias e os critérios de aceite do enunciado. Os documentos foram escritos de baixo para cima, das decisões para a consolidação, e cada etapa usou as anteriores como insumo:

| Ordem | Documento | Por que nessa posição |
| --- | --- | --- |
| 1 | ADRs | As decisões são a base de tudo. Fechá-las primeiro, isoladas e com origem na transcrição, evita que os outros documentos inventem decisões |
| 2 | RFC | Consolida a proposta técnica em cima dos ADRs já escritos. As alternativas descartadas e as questões em aberto da reunião têm lugar natural aqui |
| 3 | FDD | Com as decisões formalizadas, o desenho de implementação se constrói em cima delas, junto com a seção obrigatória "Integração com o sistema existente" |
| 4 | PRD | É o documento mais alto nível. Com RFC, FDD e ADRs em mãos, virou praticamente uma consolidação |
| 5 | Tracker | Escrito por último, como verificação: cada item dos documentos é cruzado com a fala ou o arquivo de origem |

Como organizei a interação com a IA:

- **Uma sessão por documento.** Assim o contexto de cada sessão ficava focado e os commits ficavam separados por etapa.
- **Prompts com o enunciado embutido.** Cada prompt reproduzia as seções obrigatórias e os critérios de aceite, para a IA não depender de interpretação própria sobre o que precisava entregar.
- **Alerta explícito sobre o que não entra.** Desde o primeiro prompt, avisei que a reunião descarta ideias e que elas não podiam virar requisito.
- **Revisão manual de cada entrega.** Eu lia cada documento gerado, apontava os problemas em prompts curtos e direcionados e, quando era mais rápido, editava direto no arquivo.
- **Commit ao fim de cada etapa.** Cada documento revisado era commitado antes de passar ao próximo. O RFC e a revisão do FDD foram feitos em branch e mesclados na `main`.

## Prompts customizados

Os prompts abaixo foram transcritos das sessões. Para facilitar a leitura, corrigi a digitação e reorganizei cada um em contexto, tarefa e critérios numerados, sem mudar o que foi pedido.

**1. Exploração inicial e ADRs** (primeira sessão). Além de reproduzir os requisitos dos ADRs, o prompt alerta sobre o que foi descartado na reunião e define que as decisões secundárias ficam no FDD:

```text
Contexto:
Faça uma exploração inicial para entender estrutura, padrões e o que a feature precisa endereçar.
Nesse projeto vou precisar que crie um pacote de documentação: PRD, FDD, RFC, Tracker e os ADRs.
Quero começar pelos ADRs; as instruções dos outros documentos virão depois. A reunião está no
arquivo TRANSCRICAO.md.

Tarefa:
Produza entre 5 e 8 ADRs em arquivos separados dentro de docs/adrs/.

Critérios:
1. Nome dos arquivos no formato ADR-NNN-titulo-em-kebab-case.md (ex: ADR-001-outbox-no-mysql.md).
2. Formato MADR (ou variante padrão) com, no mínimo, as seções:
   2.1. Status
   2.2. Contexto
   2.3. Decisão
   2.4. Alternativas Consideradas (pelo menos 1 alternativa real discutida ou plausível)
   2.5. Consequências (positivas e negativas, com trade-off explícito)
3. Pelo menos 1 ADR deve referenciar explicitamente arquivos, módulos ou padrões do código
   existente.
4. O conjunto deve cobrir, no mínimo, 5 das 6 decisões principais discutidas na reunião:
   4.1. Padrão Outbox no MySQL
   4.2. Política de retry com backoff e DLQ
   4.3. Autenticação HMAC-SHA256 com secret por endpoint
   4.4. Garantia at-least-once com X-Event-Id
   4.5. Worker em processo separado em polling
   4.6. Reuso dos padrões existentes do projeto
5. Decisões técnicas secundárias (formato de payload, timeouts, headers, entre outras) podem virar
   ADRs adicionais ou ficar apenas no FDD; neste caso, prefiro que fiquem no FDD.

Atenção:
Tenha cuidado com o que NÃO entra na documentação. A reunião descarta explicitamente algumas
ideias. Se elas aparecerem como requisito nos documentos criados, é sinal de que a IA não está
sendo cuidadosa com o que estou pedindo.
```

**2. RFC consolidado sobre os ADRs.** O prompt amarra o RFC aos ADRs já escritos e delimita a fronteira com o FDD, para evitar detalhe de implementação duplicado:

```text
Contexto:
Com os ADRs feitos, consolide a proposta técnica em cima das decisões. As alternativas
descartadas e as questões em aberto da reunião têm lugar natural aqui. O RFC opera em nível de
arquitetura: apresenta a abordagem escolhida, as alternativas que foram colocadas na mesa e as
questões deixadas em aberto.

Tarefa:
Escreva no arquivo docs/RFC.md a proposta técnica da solução, no formato de um documento
submetido à equipe para revisão, referenciando os ADRs já escritos.

Seções obrigatórias:
1. Metadados (autor, status, data, revisores); use os participantes da reunião como revisores.
2. Resumo executivo (TL;DR) da proposta.
3. Contexto e problema.
4. Proposta técnica (visão geral da solução, sem descer ao detalhe de implementação do FDD).
5. Alternativas consideradas (pelo menos 2 alternativas reais discutidas e descartadas na
   reunião, cada uma com o trade-off que levou ao descarte).
6. Questões em aberto (pelo menos 2 pontos levantados na reunião e não decididos ou adiados).
7. Impacto e riscos.
8. Decisões relacionadas (links para os ADRs correspondentes).

Critérios:
1. Documento conciso, de 2 a 4 páginas.
2. Não duplicar o detalhamento do FDD: o RFC responde "o que propomos e por quê"; o "como
   construir" em detalhe fica no FDD, que será criado depois.
```

**3. FDD sobre as decisões formalizadas.** O prompt lista as seções mínimas do FDD e reforça a seção obrigatória "Integração com o sistema existente", exigindo caminhos de arquivo reais do código base:

```text
Contexto:
Com as decisões formalizadas e a proposta consolidada, o desenho técnico se constrói em cima
delas. O FDD é o documento mais técnico e precisa estar acionável o suficiente para um
desenvolvedor pegar e começar a codar.

Tarefa:
Escreva no arquivo docs/FDD.md o "como implementar" da feature.

Seções obrigatórias:
1. Contexto e motivação técnica.
2. Objetivos técnicos.
3. Escopo e exclusões.
4. Fluxos detalhados (criação do evento na outbox, processamento pelo worker, retry, DLQ).
5. Contratos públicos (endpoints HTTP com payloads de exemplo, headers, status codes, semântica).
6. Matriz de erros previstos, com códigos no padrão WEBHOOK_*.
7. Estratégias de resiliência (timeouts, retries, backoff, fallback).
8. Observabilidade (métricas, logs, tracing).
9. Dependências e compatibilidade.
10. Critérios de aceite técnicos.
11. Riscos e mitigação.
12. Integração com o sistema existente (seção obrigatória adicional).

Critérios da seção "Integração com o sistema existente":
1. Nomear pelo menos 4 caminhos de arquivo reais do código base.
2. Descrever como o módulo de webhooks vai se integrar com cada um (por exemplo, como o método
   changeStatus será estendido, como as classes de erro existentes serão reutilizadas).
```

**4. Revisão do FDD.** Exemplo de prompt curto de correção, escrito depois da leitura do documento gerado:

```text
Contexto:
Verifiquei um excesso de referências com horário e nome da reunião no FDD.

Tarefa:
1. Revise o documento e retire o excesso dessas referências.
2. Aplique um cabeçalho com as informações do documento, incluindo o status.
```

**5. PRD como consolidação.** O prompt parte do RFC, do FDD e dos ADRs já escritos e traz os critérios de aceite do PRD (requisitos funcionais, meta quantitativa, riscos e itens fora de escopo):

```text
Contexto:
O PRD é o documento mais alto nível. Com RFC, FDD e ADRs em mãos, ele vira praticamente uma
consolidação.

Tarefa:
Escreva no arquivo docs/PRD.md a cobertura da feature Sistema de Webhooks de Notificação de
Pedidos.

Seções obrigatórias:
1. Resumo e contexto da feature.
2. Problema e motivação.
3. Público-alvo e cenários de uso.
4. Objetivos e métricas de sucesso.
5. Escopo (incluso e fora de escopo).
6. Requisitos funcionais.
7. Requisitos não funcionais.
8. Decisões e trade-offs principais.
9. Dependências.
10. Riscos e mitigação.
11. Critérios de aceitação.
12. Estratégia de testes e validação.

Critérios:
1. Identificar no mínimo 8 requisitos funcionais discutidos na reunião.
2. Incluir pelo menos 1 objetivo com métrica e meta quantitativa.
3. Na seção "Fora de escopo", listar explicitamente pelo menos 2 itens descartados ou adiados
   durante a reunião (TRANSCRICAO.md).
4. Na seção "Riscos", incluir pelo menos 2 riscos com probabilidade, impacto e mitigação.
```

**6. Tracker como verificação contra alucinações.** O prompt define o formato obrigatório da tabela e as metas de cobertura, que depois foram conferidas por script:

```text
Contexto:
O tracker funciona como uma referência cruzada: permite que qualquer leitor entenda de onde veio
cada decisão, requisito ou restrição, e garante que a documentação está alinhada com o que foi
efetivamente discutido e com o que existe no código.

Tarefa:
No arquivo docs/TRACKER.md, crie uma tabela markdown que mapeia cada item registrado nos
documentos à sua origem na transcrição ou no código.

Formato obrigatório da tabela:
ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização

Onde:
1. ID: identificador único do item (ex: PRD-FR-01, RFC-ALT-02, FDD-CONTRATO-03, ADR-002).
2. Documento: arquivo onde o item aparece.
3. Tipo: Requisito Funcional, Requisito Não Funcional, Decisão, Restrição, Trade-off, entre
   outros.
4. Conteúdo (resumo): descrição de uma linha do item.
5. Fonte: TRANSCRICAO ou CODIGO.
6. Localização: para TRANSCRICAO, timestamp + nome do falante (ex: [09:17] Diego); para CODIGO,
   caminho do arquivo.

Critérios de cobertura:
1. Pelo menos 80% dos itens identificáveis nos documentos têm linha correspondente no tracker.
2. Pelo menos 70% das linhas têm Fonte = TRANSCRICAO com timestamp válido no formato [hh:mm] Nome.
3. Pelo menos 5 linhas têm Fonte = CODIGO com caminho de arquivo real.
```

## Iterações e ajustes

Foram **6 iterações principais** até o resultado final: uma por documento, três delas com correções depois da primeira versão, e uma verificação final do pacote completo.

1. **ADRs.** A primeira versão cobriu as 6 decisões obrigatórias e mais duas (ordenação por `order_id` e snapshot do payload), totalizando 8 ADRs. Os erros só apareceram na iteração 5: a verificação cruzada do tracker mostrou que a IA tinha atribuído falas a minutos ou pessoas errados. Os erros corrigidos:
   - **ADR-001:** o índice da outbox estava em `[09:07] Diego`; a fala é de `[09:08]`.
   - **ADR-002:** o `PrismaClient` por processo estava em `[09:11]-[09:12] Bruno/Diego`; a fala é de `[09:30] Bruno`.
   - **ADR-003:** a proposta de 3 tentativas estava em `[09:15] Bruno`; a fala é de `[09:16]`.
   - **ADR-004:** o ADR dizia que Sofia citou Stripe e GitHub ao falar de HMAC. Quem citou foi Diego, e falando de at-least-once.
   - **ADR-003:** o trade-off falava em "quase um dia" de retentativas, contradizendo as ~15 horas decididas na reunião.

   Esse é o tipo de alucinação que passa despercebido numa leitura normal e que o tracker existe para pegar.
2. **RFC.** Produzido em branch própria a partir dos ADRs, foi mesclado na `main` sem correções de conteúdo.
3. **FDD.** Foi o documento que mais exigiu revisão:
   - **Primeira revisão:** pedi para retirar as autorreferências `[FDD]` repetidas em vários capítulos, reduzir o excesso de citações `[hh:mm] Nome` (que poluíam a leitura de um documento de implementação) e incluir um cabeçalho de metadados com autor, status e revisores.
   - **Seção "Definições deste FDD a confirmar":** removi à mão essa seção 14 gerada pela IA, uma tabela de 14 pontos a validar que, na prática, repetia decisões já descritas no corpo do documento. Removi também a nota "estimativas do autor" da tabela de riscos.
   - **Referências órfãs:** pedi à IA para retirar as menções que ainda apontavam para a seção 14 ("a confirmar (seção 14)") na tabela de retry e nos riscos.
4. **PRD.** Gerado como consolidação dos três documentos anteriores, com os critérios de aceite do enunciado no prompt (mínimo de 8 requisitos funcionais, objetivo com meta quantitativa, riscos com probabilidade, impacto e mitigação). Não precisou de correção de conteúdo.
5. **Tracker.** Além de revelar os erros dos ADRs descritos na iteração 1, passou por dois ajustes:
   - **Itens sem origem:** a IA separou numa seção própria as definições de design que não vêm nem da reunião nem do código (como o lote de 20 eventos e a assinatura dupla durante a rotação de secret), em vez de atribuir a elas uma fonte falsa.
   - **Leitura:** ao revisar, achei confusa a tabela única com 276 linhas e pedi para dividi-la em uma tabela por documento (PRD, RFC, FDD e uma por ADR).

   As metas foram conferidas por script: 85,5% das linhas com fonte na transcrição, 40 linhas com fonte no código, todos os caminhos de arquivo existentes e todos os pares `[hh:mm] Nome` presentes na transcrição.
6. **Verificação final.** Com o pacote pronto, passei para a IA os critérios de aceite de cada documento, um por vez, e pedi um checklist com a evidência de cada item, conferida contra a transcrição e o código. PRD, RFC, ADRs, tracker e este README passaram sem correção. Os ajustes foram:
   - **FDD:** na seção "Contratos públicos", só o `POST /webhooks` tinha exemplo completo de request e response. Foram acrescentados a resposta do `PATCH /webhooks/:id` e exemplos de requisição para listagem, rotação de secret, histórico de entregas e replay da DLQ, totalizando 6 endpoints completos.
   - **`docs/adrs/README.md`:** o arquivo, herdado do repositório base, indicava o padrão de nome `0001-titulo-da-decisao.md`, que contradizia o formato `ADR-NNN-titulo-em-kebab-case.md` usado nos ADRs. O padrão foi corrigido.
   - **README:** os prompts do FDD e do PRD, que tinham ficado de fora, foram incluídos na seção "Prompts customizados".

## Como navegar a entrega

```
.
├── README.md                 este documento (processo de produção)
├── TRANSCRICAO.md            transcrição da reunião (fonte, não alterada)
├── docs/
│   ├── PRD.md                por que e o quê: problema, escopo, requisitos, métricas
│   ├── RFC.md                proposta técnica, alternativas descartadas, questões em aberto
│   ├── FDD.md                como construir: fluxos, contratos, erros, integração com o código
│   ├── TRACKER.md            origem de cada item na transcrição ou no código
│   └── adrs/
│       ├── ADR-001-outbox-no-mysql.md
│       ├── ADR-002-worker-dedicado-em-polling.md
│       ├── ADR-003-retry-com-backoff-e-dead-letter-queue.md
│       ├── ADR-004-hmac-sha256-com-secret-por-endpoint.md
│       ├── ADR-005-garantia-at-least-once-com-event-id.md
│       ├── ADR-006-reuso-dos-padroes-existentes-do-projeto.md
│       ├── ADR-007-ordenacao-por-order-id-sem-garantia-global.md
│       └── ADR-008-snapshot-do-payload-na-insercao-do-outbox.md
└── src/, prisma/, tests/     código da aplicação (contexto, não alterado)
```

A ordem de leitura sugerida vai do problema ao detalhe, que é o inverso da ordem de produção:

1. **[PRD](docs/PRD.md):** entender o problema dos clientes, o escopo e o que ficou de fora.
2. **[RFC](docs/RFC.md):** ver a solução proposta e por que as alternativas foram descartadas.
3. **[ADRs](docs/adrs/):** aprofundar cada decisão, na ordem numérica.
4. **[FDD](docs/FDD.md):** especificação para implementar, incluindo a seção "Integração com o sistema existente".
5. **[Tracker](docs/TRACKER.md):** consultar a origem de qualquer item ao longo da leitura.
