+++
author = "girorme"
title = "Por que o Apache Kafka existe?"
date = "2026-10-29"
description = "Entenda por que o Apache Kafka foi criado, quais problemas ele resolve e como surgiu o event streaming."
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

Uma compra foi aprovada no seu e-commerce. E agora?

O estoque precisa reservar os produtos. O time de antifraude quer analisar a transação. A transportadora precisa preparar a entrega, o cliente espera uma notificação e alguém do analytics certamente vai querer esses dados.

É um único pedido, mas vários sistemas precisam saber que ele existe.

No começo parece simples. O serviço de pedidos chama o estoque, depois chama o pagamento e por último manda um e-mail. Tudo síncrono, fácil de desenhar e fácil de explicar naquela reunião de arquitetura.

Aí o sistema cresce.

Chega um novo consumidor, depois outro, um deles fica fora do ar, outro precisa reprocessar os pedidos da semana passada e pronto: aquele fluxo simples virou uma coleção de integrações, retentativas e scripts que ninguém quer encostar na sexta-feira.

Kafka nasceu para resolver uma versão bem grande desse problema.

Antes de falar sobre Partitions, Offsets, Replication e outras coisas que normalmente aparecem logo no primeiro tutorial, quero começar pelo básico: por que alguém precisou criar o Kafka?

## Uma compra, vários sistemas e um problema

Vamos continuar com o e-commerce.

O serviço de pedidos pode chamar cada sistema diretamente por HTTP. Isso funciona e, dependendo do caso, pode ser exatamente a solução certa. Nem toda integração precisa de um cluster distribuído no meio.

O problema aparece quando cada novo uso do dado cria uma conexão nova:

- pedidos chama estoque;
- pedidos chama pagamento;
- pedidos chama notificações;
- pedidos envia dados para analytics;
- pedidos atualiza o CRM;
- pedidos precisa avisar algum sistema que ainda nem existe.

O serviço começa a conhecer detalhes demais. Se o CRM estiver fora do ar, a compra falha? Se analytics estiver lento, o cliente espera? Onde salvamos o que não foi entregue? Como um sistema novo recupera os pedidos antigos?

Essas perguntas acabam virando código dentro de quem produz o dado. O serviço de pedidos deixa de cuidar só de pedidos e passa a administrar a vida de todos os consumidores.

Podemos colocar uma infraestrutura de mensageria no meio e desacoplar as duas pontas. Beleza. Mas alguns cenários pedem algo além de entregar uma mensagem e seguir em frente.

Um consumidor pode querer processar o evento agora. Outro só durante a madrugada. Um terceiro pode surgir daqui a três meses e precisar reler o histórico. Se algum deles ficar indisponível, o produtor não deveria parar por causa disso.

É aqui que o modelo de log do Kafka começa a fazer sentido.

## Evento não é só mais um JSON

Quando pensamos em dados, normalmente pensamos no estado atual de alguma coisa: o saldo da conta, o status do pedido, a quantidade disponível no estoque.

Um evento conta outra história. Ele registra algo que aconteceu em um momento específico:

- `PedidoConfirmado`;
- `PagamentoAprovado`;
- `SaldoDebitado`;
- `ItemReservado`.

A [documentação do Apache Kafka](https://kafka.apache.org/intro) define evento como o registro de que alguma coisa aconteceu. Dentro do Kafka, esse registro normalmente tem uma key, um value, um timestamp e pode carregar headers.

Mas o formato é a parte menos interessante agora.

O ponto é que `PedidoConfirmado` representa um fato. O estoque pode reagir a ele. O sistema de notificações também. Analytics pode ler o mesmo evento sem que o serviço de pedidos precise conhecer nenhum desses detalhes.

Se amanhã aparecer um novo consumidor, o produtor não precisa ganhar mais um `if`, uma chamada HTTP e uma política de retry.

Claro que isso não remove os problemas de sistemas distribuídos. Duplicidade, ordenação, contratos, falhas e mudanças de schema continuam existindo. Kafka não joga uma nuvem mágica em cima da arquitetura xD.

O que muda é o lugar onde lidamos com esses problemas. Em vez de espalhá-los por várias integrações ponto a ponto, criamos um fluxo de eventos que pode ser compartilhado.

## O problema do LinkedIn

Kafka começou dentro do LinkedIn.

No fim dos anos 2000, a empresa produzia cada vez mais dados de atividade: pageviews, pesquisas, anúncios exibidos, métricas de serviços e logs. Parte desses dados precisava chegar ao Hadoop e ao data warehouse para processamento offline. Outra parte precisava alimentar sistemas online e monitoramento com baixa latência.

O texto [*A Brief History of Scaling LinkedIn*](https://www.linkedin.com/blog/engineering/architecture/brief-history-scaling-linkedin) explica que a empresa tinha várias pipelines customizadas para tracking, filas, logs e atualização de sistemas. Cada pipeline resolvia um pedaço do problema. Cada uma também precisava ser escalada, corrigida e operada.

Em algum momento a conta deixou de fechar.

Jay Kreps, Neha Narkhede e Jun Rao desenvolveram Kafka como uma infraestrutura comum baseada na ideia de commit log. O LinkedIn [abriu o código em janeiro de 2011](https://www.linkedin.com/blog/member/archive/open-source-linkedin-kafka), apresentando o projeto como uma fila distribuída e persistente para eventos de atividade.

Poucos meses depois, a versão 0.6 já era descrita como um sistema publish/subscribe usado para logs, activity streams e consumo online e offline. Em julho de 2011, Kafka entrou no Apache Incubator.

O [paper original, apresentado no NetDB 2011](https://cwiki.apache.org/confluence/download/attachments/27822226/Kafka-netdb-06-2011.pdf), mostra bem a intenção: juntar o throughput dos agregadores de log, uma API parecida com sistemas de mensageria e a capacidade de processar eventos em tempo real.

Kafka não nasceu simplesmente como “uma fila mais rápida”. A ideia era substituir várias pipelines específicas por um fluxo comum de dados.

## Então... o que Kafka faz?

Se eu tivesse que explicar Kafka sem entrar na arquitetura interna, usaria três peças:

**Producer** é quem publica eventos.

**Topic** é onde eventos relacionados são organizados. Podemos ter, por exemplo, um Topic chamado `pedidos-confirmados`.

**Consumer** é quem lê e processa esses eventos.

Entre eles existe o cluster Kafka. Ele recebe os eventos e os mantém disponíveis de acordo com a política de retenção configurada.

Isso já muda bastante o desenho.

O Producer não precisa saber quantos Consumers existem. Também não precisa esperar que todos processem o evento. Cada Consumer segue no próprio ritmo e pode ter um objetivo completamente diferente.

Um atualiza o estoque. Outro alimenta o data lake. Outro calcula métricas. Todos partem do mesmo fluxo.

E a leitura não precisa destruir o evento. Enquanto o dado estiver retido, outros consumidores podem lê-lo e consumidores existentes podem voltar no histórico. Como Kafka organiza e distribui esse armazenamento fica para os próximos artigos. Por enquanto, a ideia principal é esta: armazenar faz parte do modelo, não é um detalhe escondido.

## Event streaming sem complicar

O nome parece mais complicado do que a ideia.

Event streaming é trabalhar com um fluxo contínuo de fatos. Aplicações, bancos, dispositivos e serviços produzem eventos. Esses eventos são armazenados, processados e enviados para quem precisa deles.

Alguns consumidores reagem na hora. Outros trabalham depois.

No exemplo da compra, o antifraude pode analisar o pagamento assim que ele acontece. Mais tarde, uma versão nova do modelo pode reler eventos antigos para testar outro comportamento. Analytics usa o mesmo fluxo sem obrigar o serviço de pedidos a manter uma integração separada.

Kafka junta três capacidades nesse processo:

1. publicar e assinar streams de eventos;
2. armazenar esses eventos de forma durável;
3. processá-los agora ou reler o que ficou armazenado.

É essa combinação que faz Kafka ser chamado de plataforma de event streaming.

## Kafka é uma fila?

A resposta curta: ele pode resolver problemas de mensageria, mas pensar nele apenas como fila esconde boa parte do que o torna diferente.

Em uma work queue tradicional, a mensagem geralmente representa um trabalho que deve ser entregue a algum consumidor. Esse modelo costuma oferecer recursos como roteamento, prioridade e acknowledgements individuais.

Kafka coloca um log retido no centro. Consumidores acompanham a própria posição nesse log, vários grupos podem ler o mesmo evento e o histórico pode ser relido enquanto estiver disponível.

Isso não quer dizer que Kafka é “melhor” do que RabbitMQ, ActiveMQ, SQS ou qualquer outro broker. Essa comparação quase sempre depende do problema.

Precisa de roteamento complexo, prioridade entre tarefas e uma fila de trabalho curta? Um broker orientado a filas pode ser mais simples.

Precisa manter um histórico, permitir replay, alimentar vários consumidores e lidar com um fluxo contínuo de eventos? O modelo do Kafka começa a ficar interessante.

Também não dá pra fingir que todos os message brokers trabalham da mesma forma. Muitos já oferecem persistência, múltiplos padrões de entrega e recursos de streaming. A arquitetura deve partir da semântica necessária, não do logo mais popular no diagrama.

## Isso funciona fora dos slides?

No LinkedIn, os primeiros casos envolviam activity tracking, métricas, logs e envio de dados para sistemas analíticos. O mesmo fluxo atendia consumidores online e processamento offline. Era justamente o problema que a empresa queria resolver.

A Uber publicou outro exemplo em 2020. No artigo [*Disaster Recovery for Multi-Region Kafka at Uber*](https://www.uber.com/blog/kafka/), a equipe descreve o uso de Kafka como barramento pub/sub para eventos dos aplicativos, fonte para streaming analytics, transporte de changelogs de bancos e ingestão no data lake.

O texto também mostra a parte que costuma ficar fora dos tutoriais: quando Kafka vira infraestrutura crítica, disaster recovery, replicação entre regiões e operação passam a fazer parte do trabalho.

Não é só subir um container e mandar alguns eventos. Em produção sempre chega a conta xD.

## Quando Kafka faz sentido?

Não existe uma regra única, mas alguns sinais aparecem bastante:

- vários sistemas precisam reagir aos mesmos eventos;
- cada consumidor trabalha em um ritmo diferente;
- replay e reconstrução de estado têm valor real;
- o volume é contínuo e pode crescer;
- consumidores podem ficar fora do ar sem parar os produtores;
- sistemas online e offline precisam partir da mesma fonte de dados.

Agora o outro lado.

Se existe um produtor, um consumidor, pouco volume e nenhuma necessidade de replay, Kafka pode ser uma solução grande demais. Uma chamada HTTP, uma fila gerenciada ou até o banco que já existe talvez resolvam o problema com menos peças.

Kafka traz decisões sobre retenção, capacidade, contratos, segurança, observabilidade e recuperação. Se a equipe não precisa dessas capacidades ou não consegue operar a plataforma, o ganho arquitetural desaparece rápido.

Outro cuidado: usar Kafka não transforma automaticamente o evento na fonte definitiva da verdade. Em alguns sistemas ele cumpre esse papel. Em outros, transporta mudanças e o estado autoritativo continua em um banco de dados.

Essa escolha é de arquitetura e depende das garantias que o domínio precisa.

## É isso...

Kafka surgiu porque mover dados entre muitos sistemas, com baixa latência e possibilidade de reprocessamento, ficou difícil demais usando várias pipelines isoladas.

A proposta foi criar um log compartilhado. Producers publicam fatos, Kafka mantém o fluxo e Consumers leem sem exigir uma integração exclusiva com quem produziu o dado.

Ainda tem bastante coisa escondida nesse desenho. No próximo artigo vamos entrar em eventos, Producers, Consumers e Topics com mais calma. Depois chegamos em Partitions, Offsets, Replication e toda a diversão de sistemas distribuídos :)

## Referências

- Apache Kafka. [*Introduction*](https://kafka.apache.org/intro). Acesso em 29 out. 2026.
- Confluent Developer. [*What You'll Learn in This Apache Kafka 101 Course*](https://developer.confluent.io/courses/apache-kafka/events/). Acesso em 29 out. 2026.
- Kreps, Jay; Narkhede, Neha; Rao, Jun. [*Kafka: a Distributed Messaging System for Log Processing*](https://cwiki.apache.org/confluence/download/attachments/27822226/Kafka-netdb-06-2011.pdf). NetDB, 2011.
- Rao, Jun. [*Open-sourcing Kafka, LinkedIn's distributed message queue*](https://www.linkedin.com/blog/member/archive/open-source-linkedin-kafka). LinkedIn, 11 jan. 2011.
- Narkhede, Neha. [*Project Kafka, a distributed publish-subscribe messaging system, reaches v0.6*](https://www.linkedin.com/blog/engineering/open-source/project-kafka-distributed-publish-subscribe-messaging-system-reaches-v06). LinkedIn, 16 jun. 2011.
- LinkedIn Engineering. [*A Brief History of Scaling LinkedIn*](https://www.linkedin.com/blog/engineering/architecture/brief-history-scaling-linkedin). 2015.
- Fu, Yupeng; Chen, Mingmin. [*Disaster Recovery for Multi-Region Kafka at Uber*](https://www.uber.com/blog/kafka/). Uber Engineering, 21 dez. 2020.

