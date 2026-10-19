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

Os prompts abaixo foram transcritos das sessões, com pequenas correções de digitação.

**1. Exploração inicial e ADRs** (primeira sessão). Além de reproduzir os requisitos dos ADRs, o prompt alerta sobre o que foi descartado na reunião e define que as decisões secundárias ficam no FDD:

```text
Faça uma exploração inicial para entender estrutura, padrões e o que a feature precisa endereçar.
Nesse projeto vou precisar que crie um pacote de documentação, entre eles: PRD, FDD, RFC, Tracker
e os ADRs. Quero começar primeiro com os ADRs e a seguir irei passar as instruções. Produza entre
5 e 8 ADRs em arquivos separados dentro de docs/adrs/, nomeados no formato
ADR-NNN-titulo-em-kebab-case.md (ex: ADR-001-outbox-no-mysql.md). Cada ADR deve seguir o formato
MADR (ou variante padrão) com no mínimo as seções: Status, Contexto, Decisão, Alternativas
Consideradas (pelo menos 1 alternativa real discutida ou plausível), Consequências (positivas e
negativas, com trade-off explícito). Pelo menos 1 ADR deve referenciar explicitamente arquivos,
módulos ou padrões do código existente. O conjunto de ADRs deve cobrir, no mínimo, 5 das 6
decisões principais discutidas na reunião: Padrão Outbox no MySQL, Política de retry com backoff
e DLQ, Autenticação HMAC-SHA256 com secret por endpoint, Garantia at-least-once com X-Event-Id,
Worker em processo separado em polling, Reuso dos padrões existentes do projeto. Decisões técnicas
secundárias (formato de payload, timeouts, headers, entre outras) podem virar ADRs adicionais ou
ficar apenas no FDD, mas neste caso prefiro que fique no FDD. A reunião está no arquivo
TRANSCRICAO.md. Tenha cuidado com o que NÃO entra na documentação. A reunião descarta
explicitamente algumas ideias. Se essas coisas aparecerem como requisito nos documentos criados,
é sinal de que a IA não está sendo cuidadosa com o que estou pedindo.
```

**2. RFC consolidado sobre os ADRs.** O prompt amarra o RFC aos ADRs já escritos e delimita a fronteira com o FDD, para evitar detalhe de implementação duplicado:

```text
Com os ADRs feitos agora, consolide a proposta técnica em cima das decisões. As alternativas
descartadas e as questões em aberto da reunião têm lugar natural aqui. Referencie os ADRs já
escritos. Utilize o arquivo docs/RFC.md com a proposta técnica da solução, no formato de um
documento submetido à equipe para revisão. O RFC opera em nível de arquitetura: apresenta a
abordagem escolhida, as alternativas que foram colocadas na mesa e as questões deixadas em
aberto. É um documento conciso (2 a 4 páginas); o detalhamento de implementação vai ficar no FDD
que será criado depois. Deve incluir no mínimo: Metadados (autor, status, data, revisores); use os
participantes da reunião como revisores, Resumo executivo (TL;DR) da proposta, Contexto e
problema, Proposta técnica (visão geral da solução, sem descer ao detalhe de implementação do
FDD), Alternativas consideradas (pelo menos 2 alternativas reais discutidas e descartadas na
reunião, cada uma com o trade-off que levou ao descarte), Questões em aberto (pelo menos 2 pontos
levantados na reunião e não decididos ou adiados), Impacto e riscos, Decisões relacionadas (links
para os ADRs correspondentes). O RFC não deve duplicar o detalhamento do FDD. Ele responde "o que
propomos e por quê"; o "como construir" em detalhe fica no FDD que será implementado depois.
```

**3. FDD sobre as decisões formalizadas.** O prompt lista as seções mínimas do FDD e reforça a seção obrigatória "Integração com o sistema existente", exigindo caminhos de arquivo reais do código base:

```text
Com as decisões formalizadas e a proposta consolidada, o desenho técnico se constrói em cima
delas. Lembre da seção obrigatória "Integração com o sistema existente". Inclua no arquivo
docs/FDD.md detalhando o "como implementar" da feature. O FDD é o documento mais técnico e precisa
estar acionável o suficiente para um desenvolvedor pegar e começar a codar. Deve ter no mínimo:
Contexto e motivação técnica, Objetivos técnicos, Escopo e exclusões, Fluxos detalhados (criação
do evento na outbox, processamento pelo worker, retry, DLQ), Contratos públicos (endpoints HTTP
com payloads de exemplo, headers, status codes, semântica), Matriz de erros previstos com códigos
no padrão WEBHOOK_*, Estratégias de resiliência (timeouts, retries, backoff, fallback),
Observabilidade (métricas, logs, tracing), Dependências e compatibilidade, Critérios de aceite
técnicos, Riscos e mitigação. Seção obrigatória adicional: "Integração com o sistema existente".
Esta seção deve nomear pelo menos 4 caminhos de arquivo reais do código base e descrever como o
módulo de webhooks vai se integrar com cada um (por exemplo, como o método changeStatus será
estendido, como as classes de erro existentes serão reutilizadas).
```

**4. Revisão do FDD.** Exemplo de prompt curto de correção, escrito depois da leitura do documento gerado:

```text
Verifiquei também um excesso de referências com horário e nome da reunião. Revise o documento e
retire o excesso dessas referências. Aplique também um cabeçalho com as informações do documento,
status.
```

**5. PRD como consolidação.** O prompt parte do RFC, do FDD e dos ADRs já escritos e traz os critérios de aceite do PRD (requisitos funcionais, meta quantitativa, riscos e itens fora de escopo):

```text
Como o PRD é mais alto nível, com RFC, FDD e ADRs em mãos vira praticamente uma consolidação. No
arquivo docs/PRD.md faça a cobertura da feature de Sistema de Webhooks de Notificação de Pedidos.
O PRD deve ter no mínimo as seguintes seções: Resumo e contexto da feature, Problema e motivação,
Público-alvo e cenários de uso, Objetivos e métricas de sucesso, Escopo (incluso e fora de
escopo), Requisitos funcionais, Requisitos não funcionais, Decisões e trade-offs principais,
Dependências, Riscos e mitigação, Critérios de aceitação e Estratégia de testes e validação. A
seção "Fora de escopo" deve listar explicitamente pelo menos 2 itens descartados ou adiados
durante a reunião (TRANSCRICAO.md). Aplique os seguintes critérios: identifica no mínimo 8
requisitos funcionais discutidos na reunião, inclui pelo menos 1 objetivo com métrica e meta
quantitativa e, na seção "Riscos", inclui pelo menos 2 riscos com probabilidade, impacto e
mitigação.
```

**6. Tracker como verificação contra alucinações.** O prompt define o formato obrigatório da tabela e as metas de cobertura, que depois foram conferidas por script:

```text
No arquivo docs/TRACKER.md crie uma tabela markdown que mapeia cada item registrado nos documentos
à origem na transcrição ou no código. O tracker funciona como uma referência cruzada: permite que
qualquer leitor entenda de onde veio cada decisão, requisito ou restrição, e garante que a
documentação está alinhada com o que foi efetivamente discutido e com o que existe no código.
Formato obrigatório da tabela:
ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização
Onde: ID: identificador único do item (ex: PRD-FR-01, RFC-ALT-02, FDD-CONTRATO-03, ADR-002),
Documento: arquivo onde o item aparece, Tipo: Requisito Funcional, Requisito Não Funcional,
Decisão, Restrição, Trade-off, entre outros, Conteúdo (resumo): descrição de uma linha do item,
Fonte: TRANSCRICAO ou CODIGO, Localização: para TRANSCRICAO, timestamp + nome do falante
(ex: [09:17] Diego). Para CODIGO, caminho do arquivo. Cobertura mínima: pelo menos 80% dos itens
identificáveis nos seus documentos devem ter linha correspondente no tracker. Pelo menos 70% das
linhas têm Fonte = TRANSCRICAO com timestamp válido no formato [hh:mm] Nome. Pelo menos 5 linhas
têm Fonte = CODIGO com caminho de arquivo real.
```

## Iterações e ajustes

Foram **5 iterações principais** até o resultado final, uma por documento, e três delas exigiram correções depois da primeira versão.

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
