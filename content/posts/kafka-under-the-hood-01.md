+++
author = "girorme"
title = "Por que o Apache Kafka existe?"
date = "2026-10-29"
description = "Entenda por que o Apache Kafka foi criado, quais problemas ele resolve e como surgiu o event streaming."
draft = true
tags = [
    "kafka",
    "event-streaming",
    "distributed-systems"
]
categories = [
    "kafka",
    "software-architecture"
]
+++

Uma compra acaba de ser confirmada em um e-commerce. Para o cliente, é apenas uma tela de sucesso. Dentro da empresa, esse fato pode iniciar uma pequena avalanche: o estoque precisa reservar itens, o pagamento deve ser conciliado, a transportadora precisa receber uma solicitação, o sistema antifraude atualiza seus modelos, o time de atendimento quer enxergar o pedido e a área de dados quer medir a conversão.

O problema não é executar uma dessas tarefas. É fazer com que o mesmo acontecimento chegue a vários sistemas, em tempos diferentes, sem transformar cada aplicação em uma central de integrações. Quanto mais produtos, times e bancos de dados entram na equação, mais difícil fica mover dados de maneira confiável.

Foi para uma versão desse problema — em uma escala muito maior — que o Apache Kafka nasceu. Antes de falar em Partitions, Offsets ou Replication, vale entender a necessidade que deu origem a ele.

## Quando cada sistema precisa conversar com todos os outros

Uma integração começa de forma inocente. O serviço de pedidos chama o estoque por HTTP. Depois passa a chamar também faturamento e notificações. A equipe de analytics pede uma cópia dos dados em seu warehouse. Surge um processo separado para exportar logs. Em seguida, o antifraude precisa receber as mesmas mudanças quase em tempo real.

Cada nova ligação traz seu próprio contrato, tratamento de falha, política de retentativa e ritmo de processamento. Se um destino ficar indisponível, o produtor espera, descarta, persiste para tentar depois ou bloqueia a operação? Quando um consumidor novo aparece, é possível recuperar os acontecimentos antigos? Quem garante que a versão enviada ao warehouse representa o mesmo fato visto pelo sistema operacional?

Isso não significa que chamadas diretas sejam ruins. Uma consulta que precisa de resposta imediata continua combinando bem com request/response. O problema aparece quando fatos de negócio precisam ser distribuídos para muitos interessados independentes. Conexões ponto a ponto fazem o produtor conhecer demais sobre cada destino e multiplicam caminhos que precisam ser operados.

Uma saída é colocar uma infraestrutura intermediária entre quem produz e quem consome a informação. Mas, para grandes fluxos de dados, não basta encaminhar uma mensagem e esquecê-la. Diferentes consumidores precisam ler no próprio ritmo; alguns trabalham em tempo real, outros em lotes; sistemas podem ficar fora do ar; um algoritmo novo pode precisar reprocessar o passado. É aí que a ideia de um log durável se torna importante.

## Um evento é um fato, não apenas o estado atual

Aplicações tradicionais costumam representar **coisas**: a linha do pedido, o saldo atual da conta, a quantidade disponível de um produto. Um evento representa algo que ocorreu em um momento específico: `PedidoConfirmado`, `PagamentoAprovado`, `SaldoDebitado` ou `ItemReservado`.

A [documentação do Apache Kafka](https://kafka.apache.org/intro) descreve um evento como o registro de que algo aconteceu. Em Kafka, ele normalmente carrega uma key, um value, um timestamp e, opcionalmente, headers. O nome “mensagem” também aparece, mas pensar em fatos ajuda a desenhar melhores contratos.

Considere `PedidoConfirmado`. O evento não é uma ordem genérica para “atualizar todos os sistemas”. Ele registra um fato de negócio que já aconteceu. Estoque, comunicação e analytics podem reagir a esse mesmo fato sem que o serviço de pedidos conheça a lógica interna de cada um. Amanhã, um novo consumidor pode ser adicionado sem alterar o produtor.

Esse desacoplamento não elimina os desafios. Schema evolution, duplicidade, ordenação e falhas continuam existindo. A diferença é que o fluxo de eventos cria um ponto de integração explícito e compartilhado, em vez de esconder esses problemas em dezenas de conexões bilaterais.

## O problema que o LinkedIn precisava resolver

No fim dos anos 2000, o LinkedIn produzia volumes crescentes de dados de atividade: visualizações de páginas, pesquisas, anúncios apresentados, métricas de serviços e logs. Esses dados tinham destinos diferentes. Parte alimentava Hadoop e data warehouses para análise offline; outra parte precisava chegar rapidamente a aplicações e monitoramento.

Segundo o relato [*A Brief History of Scaling LinkedIn*](https://www.linkedin.com/blog/engineering/architecture/brief-history-scaling-linkedin), a empresa acumulou pipelines customizadas para streaming, filas, logs, tracking e atualização de sistemas. Cada pipeline precisava escalar e ser mantida separadamente. A resposta foi construir uma infraestrutura comum, baseada na abstração de commit log.

Kafka foi desenvolvido no LinkedIn por Jay Kreps, Neha Narkhede e Jun Rao. Em janeiro de 2011, a empresa [anunciou a abertura do código](https://www.linkedin.com/blog/member/archive/open-source-linkedin-kafka), descrevendo-o como uma fila distribuída, persistente e eficiente para rastrear eventos de atividade. Em junho daquele ano, a versão 0.6 já era apresentada como um sistema publish/subscribe voltado a activity streams, logs e consumo online e offline; em julho, o projeto entrou no Apache Incubator.

O [paper apresentado no NetDB 2011](https://cwiki.apache.org/confluence/download/attachments/27822226/Kafka-netdb-06-2011.pdf) deixa clara a combinação desejada: o alto throughput dos agregadores de log, a interface de um sistema de mensageria e a possibilidade de consumidores online processarem eventos com baixa latência. Kafka não surgiu apenas para substituir uma fila. Surgiu para unificar movimentos de dados que antes exigiam ferramentas e pipelines diferentes.

## Kafka em termos simples

Kafka é uma plataforma distribuída de event streaming. Em um primeiro mapa mental, três nomes bastam:

- Um **Producer** publica eventos.
- Um **Topic** organiza um fluxo relacionado, como `pedidos-confirmados`.
- Um **Consumer** lê e processa os eventos de que precisa.

Entre eles há um cluster Kafka, responsável por receber e manter os eventos disponíveis. Producers não precisam saber quantos Consumers existem. Consumers podem ter objetivos e velocidades distintos: um atualiza uma projeção operacional em segundos; outro transfere dados para um lake; um terceiro calcula indicadores.

O detalhe decisivo é que a leitura não precisa destruir imediatamente o evento. A retenção é configurável, e consumidores mantêm sua posição no fluxo. Isso permite que mais de uma aplicação leia o mesmo Topic e que dados ainda retidos sejam reprocessados. Como Kafka distribui e replica seus dados internamente será assunto dos próximos artigos; por enquanto, basta perceber que armazenamento não é um efeito colateral, mas parte do modelo.

## De mensagens isoladas a event streaming

**Event streaming** é a prática de capturar continuamente eventos vindos de aplicações, bancos de dados, dispositivos e outros sistemas; armazená-los de forma durável; processá-los conforme chegam ou mais tarde; e encaminhá-los aos destinos necessários.

Essa definição junta presente e passado. O antifraude pode reagir a um pagamento agora. Uma versão nova do seu modelo pode, depois, reler eventos que continuam retidos. Um pipeline pode transportar os mesmos dados para análise offline sem obrigar o sistema de origem a manter outra integração.

Kafka oferece três capacidades que sustentam esse fluxo: publicar e assinar streams de eventos, armazená-los de forma durável e processá-los no momento em que ocorrem ou retrospectivamente. É essa combinação que justifica chamá-lo de plataforma de event streaming.

## Kafka não é simplesmente “uma fila mais rápida”

Kafka e sistemas de mensageria tradicionais têm uma área de interseção: ambos desacoplam produtores e consumidores e transportam mensagens de forma assíncrona. A diferença útil está no modelo, não em uma disputa de velocidade.

Em um modelo clássico de work queue, uma mensagem costuma representar trabalho a ser entregue a um consumidor, com recursos ricos de roteamento e acknowledgements por mensagem. Em Kafka, o conceito central é um log ordenado e retido, do qual consumidores acompanham sua própria posição. O mesmo evento pode servir a vários grupos independentes e ser lido novamente enquanto estiver disponível.

Essa comparação precisa de cuidado. Message brokers modernos também oferecem persistência, replay, streams ou múltiplos padrões de entrega. Kafka, por sua vez, pode cumprir funções de mensageria. A escolha depende da semântica necessária: roteamento complexo de tarefas, filas curtas e prioridades podem favorecer um broker orientado a filas; histórico durável, múltiplos leitores e grande fluxo contínuo favorecem o modelo de log de Kafka.

## Casos que saíram do papel

No LinkedIn, os usos iniciais incluíam activity tracking, logs, métricas e carga de dados para sistemas analíticos. A infraestrutura comum permitiu alimentar tanto consumidores de baixa latência quanto processamento offline — exatamente a tensão que motivou o projeto.

A Uber documentou outro retrato em 2020. Em [*Disaster Recovery for Multi-Region Kafka at Uber*](https://www.uber.com/blog/kafka/), a equipe descreve Kafka como base para um barramento pub/sub de eventos dos aplicativos, plataformas de streaming analytics, distribuição de changelogs de bancos de dados e ingestão no data lake. O artigo também mostra o outro lado da adoção: quando Kafka vira infraestrutura crítica, recuperação de desastre, replicação entre regiões e operação passam a ser problemas de primeira classe.

Esses exemplos não são receitas para copiar. Eles demonstram uma propriedade comum: muitos produtores e consumidores precisam compartilhar fluxos de dados sem criar uma integração exclusiva para cada par.

## Quando Kafka faz sentido — e quando é excesso

Kafka é um bom candidato quando vários destes sinais aparecem juntos:

- muitos sistemas precisam reagir aos mesmos eventos;
- o fluxo é contínuo e pode crescer significativamente;
- consumidores precisam avançar em velocidades independentes;
- retenção, replay ou reconstrução de estado têm valor real;
- integrações online e offline devem partir da mesma fonte de eventos;
- falhas temporárias de consumidores não podem interromper produtores.

Ele pode ser desnecessário quando existe apenas um produtor, um consumidor e pouco volume; quando uma chamada síncrona simples expressa melhor a interação; quando o banco de dados já resolve a necessidade; ou quando a equipe não está preparada para operar mais um sistema distribuído. Kafka exige decisões sobre contratos, retenção, capacidade, segurança, observabilidade e recuperação. Adotá-lo apenas porque “event-driven” parece moderno troca um problema pequeno por uma plataforma inteira.

Também é importante não confundir Kafka com a fonte definitiva de toda verdade. Em alguns domínios ele pode ser o registro de eventos central; em outros, transporta mudanças cujo estado autoritativo permanece em um banco. Essa decisão depende das garantias e do modelo de dados da aplicação.

## O ponto de partida

Kafka existe porque mover fatos entre muitos sistemas, em tempo real e com possibilidade de reprocessamento, era um problema que pipelines isoladas e integrações ponto a ponto resolviam mal em escala. Sua resposta foi tratar eventos como um log compartilhado: Producers publicam sem conhecer cada destino, Kafka preserva o fluxo e Consumers leem de maneira independente.

Esse é apenas o mapa. No próximo artigo, vamos olhar com mais cuidado para eventos, Producers, Consumers e Topics — as peças mínimas para transformar essa ideia em um fluxo executável.

## Referências

- Apache Kafka. [*Introduction*](https://kafka.apache.org/intro). Acesso em 29 out. 2026.
- Confluent Developer. [*What You'll Learn in This Apache Kafka 101 Course*](https://developer.confluent.io/courses/apache-kafka/events/). Acesso em 29 out. 2026.
- Kreps, Jay; Narkhede, Neha; Rao, Jun. [*Kafka: a Distributed Messaging System for Log Processing*](https://cwiki.apache.org/confluence/download/attachments/27822226/Kafka-netdb-06-2011.pdf). NetDB, 2011.
- Rao, Jun. [*Open-sourcing Kafka, LinkedIn's distributed message queue*](https://www.linkedin.com/blog/member/archive/open-source-linkedin-kafka). LinkedIn, 11 jan. 2011.
- Narkhede, Neha. [*Project Kafka, a distributed publish-subscribe messaging system, reaches v0.6*](https://www.linkedin.com/blog/engineering/open-source/project-kafka-distributed-publish-subscribe-messaging-system-reaches-v06). LinkedIn, 16 jun. 2011.
- LinkedIn Engineering. [*A Brief History of Scaling LinkedIn*](https://www.linkedin.com/blog/engineering/architecture/brief-history-scaling-linkedin). 2015.
- Fu, Yupeng; Chen, Mingmin. [*Disaster Recovery for Multi-Region Kafka at Uber*](https://www.uber.com/blog/kafka/). Uber Engineering, 21 dez. 2020.

