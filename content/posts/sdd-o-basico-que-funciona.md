+++
author = "girorme"
title = "Harness engineering & SDD... o básico que funciona"
date = "2026-08-30"
description = "Callback hell? Que nada, a moda agora é: prompt e markdown hell!!! xD"
tags = [
    "llm",
    "programming",
    "ai"
]
categories = [
    "llm",
    "programming",
    "ai"
]
+++

![sdd-llm](/images/fluxo-trabalho-sdd.png)

Uma boa spec na mão ou 18 agentes alucinando? Foi o que pensei quando vi meio mundo de "especialistas" prometendo uma empresa inteira formada de agentes...

## Um panorama do momento

Se vc como eu não está em uma caverna nesse momento, trabalha / gosta de tecnologia, certamente está vivendo a "fricção cognitiva" atual de centenas de novos repositórios e temas por minuto (e tinha gente que achava que nada superava as libs de JS ~~enquanto vc lia essa frase outra lib js foi criada~~). É tanta informação, tanta coisa pra estudar, que vale acompanhar os "roadmap to roadmaps" xD. Pra te adiantar, esse artigo vai para o lado do código.

### Comportamento atual dos "codadores"

Poupando o leitor de todo aquele discurso: "Estamos em uma nova era", "LLM traz produtividade", etc, etc. O objetivo aqui é contribuir minimamente com você que, além de mandar PR para o colega de 550 arquivos, solta aquele "looks good to me" na hora de aprovar o código pra prod.

Chegamos em um momento inclusive que vemos a comunidade trabalhando para diminuir as loucuras que as LLMs podem gerar. Já deu tempo de perceber e sentir que código gerado de qualquer forma só adiciona complexidade e aumenta demais a dificuldade de manutenção.

Então, no reality show da vida tech, se formos categorizar os grupos maiores, temos:

1 - o time que gera código com LLM com prompts vagos, manda o código pra prod e vai corrigindo sem histórico, docs e "métodos"

2 - o time que "sofre menos", utilizando algum meio de trackear o código gerado por LLM e segue um processo mínimo pra diminuir a fricção

### Harness engineering

Para quem joga poker (se aplica a outros jogos...) e estuda um pouco mais a respeito, conhece o termo "GTO" que ronda as mesas profissionais e os grandes lucros. A abreviação vem de Game Theory Optimal, que visa criar uma estratégia que seja menos explorada pelos adversários. É a forma "perfeita" de jogar, uma fórmula que usa estatística e matemática visando a melhor forma de ganhar.

Assim como o GTO, harness engineering é uma estratégia "lucrativa" que se traduz em: **ambiente perfeito para a IA (LLMs) trabalhar**.

E por que é importante ter esse conceito em mente? Basicamente, o que vai ditar a qualidade do código gerado pelas LLMs é o ambiente em que elas estão sendo executadas. Isso significa que não basta escrever um prompt e esperar a mágica acontecer, pq na maioria das vezes o código gerado sem uma "armadura" frequentemente é ruim de manter e está sujeito a envelhecer da pior forma possível.

Esse **ambiente otimizado** é o harness.

### SDD, loop engineering e outras maracutaias

Quando começamos a usar agentes de IA para desenvolver software, uma coisa fica bastante clara: dizer para o agente o que fazer não é o mesmo que dizer para ele como saber que terminou. Um prompt pode ser suficiente para uma tarefa simples, mas conforme a complexidade aumenta, precisamos de algo mais estável para representar a intenção do que está sendo construído. Logo -> SDD.

A ideia é "simples": antes de partir para a implementação, definimos uma especificação que descreve o que esperamos da feature / correção de bug / uma série de outras coisas. Essa especificação passa a ser a referência durante todo o desenvolvimento, reduzindo a dependência de contexto que está apenas na conversa com o agente.

O ponto mais importante aqui é que a spec não precisa ser vista como um documento estático que escrevemos uma vez e esquecemos. Ela participa do próprio processo de desenvolvimento. Em algumas empresas já vejo inclusive times focando muito na revisão dessas specs: ela vira a fonte de verdade.

#### O loop

E quando a spec passa a ser a fonte de verdade, surge uma consequência natural: podemos deixar de pensar no agente como alguém que recebe um prompt, executa uma tarefa e espera pelo próximo comando. Podemos começar a pensar em um **loop**.

Essa ideia vem ganhando bastante espaço na comunidade de desenvolvimento com agentes. Em vez de ficarmos constantemente dizendo ao agente qual é o próximo passo, construímos um processo que consegue entender o trabalho, executar uma parte dele, verificar o resultado e decidir o que precisa acontecer em seguida. O foco deixa de estar no prompt e passa para o **sistema que conduz o agente**.

No contexto do SDD, a spec funciona como um dos pontos de referência desse loop. O agente não recebe apenas uma instrução dizendo "implemente essa feature". Ele recebe uma definição do que precisa ser verdade quando o trabalho estiver concluído.

Podemos imaginar algo relativamente simples:

> Spec → Plan → Implement → Verify → Feedback → ...

A ideia não é criar uma esteira mágica que roda sozinha. O ponto é que cada etapa deixa informação suficiente para a próxima, e a verificação consegue dizer se ainda existe trabalho a ser feito.

Essa última parte é provavelmente a mais importante. Um loop não deveria simplesmente perguntar ao próprio agente se ele "acha" que terminou. Precisamos de mecanismos externos que consigam verificar o resultado: testes, critérios de aceitação, lint, type checking, análise estática, revisão por outro agente e outras formas de validação.

Isso também muda um pouco o papel do Harness. O Harness fornece o ambiente, o contexto, as ferramentas e as regras que o agente utiliza para trabalhar. O loop utiliza esse ambiente para conduzir o trabalho de forma contínua.

Em outras palavras: **o Harness define o ambiente em que o agente trabalha; o loop define como o trabalho avança até chegar a um estado verificável.**

E é justamente aqui que a qualidade da nossa spec começa a fazer muita diferença. Quanto melhor conseguirmos transformar a intenção em critérios claros e verificáveis, mais autonomia podemos dar ao loop sem simplesmente aumentar a quantidade de prompts ou de supervisão humana.

## SPEC → PLAN → CODE → VERIFY

Antes de entrar nos detalhes, gosto de pensar nesse processo em quatro etapas:

**SPEC** define o que precisa ser verdade.

**PLAN** define como vamos fazer isso dentro do sistema existente.

**CODE** transforma o plano em implementação.

**VERIFY** tenta provar que o resultado realmente atende ao que foi definido.

Isso parece óbvio, mas a separação é importante. Se misturamos tudo em um único prompt, o agente começa a tomar decisões de arquitetura enquanto ainda está tentando entender o requisito. Quando separamos as etapas, conseguimos revisar a intenção antes da implementação e a estratégia técnica antes de escrever código.

Também não significa que cada etapa precisa ser feita por um agente diferente. O importante é separar as responsabilidades, não necessariamente as ferramentas.

### O que compõe uma boa Spec?

Uma boa spec não é simplesmente um prompt maior pedindo para a LLM implementar alguma coisa. Ela precisa fornecer contexto suficiente para entender **o problema, as restrições e principalmente o que significa terminar o trabalho corretamente**.

No meu caso, esse processo começa de forma bastante simples. A partir do Copilot, por exemplo, posso executar um `/spec` e fornecer uma task, uma necessidade de negócio ou descrever um bug. A partir daí, o agente consulta o contexto disponível e produz a especificação antes de qualquer implementação.

#### Contexto

O primeiro ponto é dar contexto para a LLM. Uma spec isolada pode descrever uma funcionalidade, mas dificilmente vai capturar as particularidades de um sistema existente.

Por isso, antes de gerar a spec, o agente pode consultar as instruções do projeto, documentação de arquitetura, regras de negócio, decisões técnicas e outros artefatos existentes.

Com MCPs, esse contexto pode ir além dos arquivos locais: podemos conectar o agente ao Jira, Notion, banco de dados, ferramentas internas, repositórios ou outras bases de conhecimento.

A ideia é simples: **quanto mais próximo o agente estiver da realidade do sistema, menos precisamos depender de suposições.**

#### Intenção e análise

Outro ponto importante é definir como queremos que o agente analise o problema. Antes de escrever a spec, ele precisa entender o problema de negócio, avaliar o impacto arquitetural, procurar ambiguidades e pensar nas possíveis falhas.

Não precisamos transformar isso em uma tentativa de reproduzir o pensamento interno da LLM. O que queremos é definir um **protocolo de análise**: quais perguntas o agente deve considerar antes de produzir a especificação.

No meu caso, isso inclui identificar suposições, separar o que está dentro e fora do escopo e dar atenção especial aos edge cases.

#### O que será feito

Depois de entender o contexto e o problema, a spec precisa deixar claro **o que será construído**.

O objetivo explica o *why*: qual problema estamos resolvendo e qual valor estamos entregando. A descrição funcional explica o *what*: qual comportamento esperamos observar quando a funcionalidade estiver pronta.

Essa separação evita que a spec fique presa a uma implementação específica. Queremos definir o resultado esperado antes de decidir exatamente quais módulos, classes ou providers serão criados.

#### Fluxo e contratos

A spec também precisa tornar o fluxo compreensível. Para uma API, podemos descrever algo como:

**Request → Validação → Dados → Regra de negócio → Persistência → Response**

Não é ainda o design detalhado da solução. É uma forma de deixar claro como a informação entra no sistema, o que esperamos que aconteça e qual deve ser o resultado.

Também entram aqui os contratos e restrições que não deveriam ficar implícitos: endpoints, eventos, modelo de dados, segurança, observabilidade e aquilo que está fora do escopo.

#### Critérios de aceitação

Talvez essa seja uma das partes mais importantes: transformar uma descrição em algo que possa ser verificado.

É aqui que entram os **critérios de aceitação**, descrevendo cenários de sucesso, erro e comportamento esperado. No meu caso, utilizo Gherkin para estruturar esses cenários de uma forma relativamente próxima do negócio.

A ideia é sair de "implementar X" e chegar em algo mais próximo de:

> **Dado** determinado contexto → **Quando** determinada ação acontece → **Então** determinado resultado deve acontecer.

A partir daí, esses critérios podem ser conectados aos testes e utilizados pelo restante do loop como referência de verificação.

#### Definition of Done

Por fim, precisamos definir o que significa **terminar**.

Código funcionando não necessariamente significa que a tarefa está concluída. Podemos exigir testes, lint, migrations, documentação, integração ou outros requisitos específicos do projeto.

A Definition of Done fecha uma parte importante do loop: fornece condições objetivas para decidir se o trabalho realmente está pronto.

## E depois da SPEC?

A SPEC não precisa carregar tudo. Na verdade, quanto mais tentamos colocar nela, maior a chance de transformar o documento em um plano de implementação disfarçado.

Depois da SPEC vem o **PLAN**.

Aqui a pergunta muda:

> **Como vamos construir isso dentro do sistema que já existe?**

_obs: Aqui em  um cenário que estamos trabalhando em um sistema que já existe, seja ele complexo ou não._

O PLAN recebe a SPEC e o codebase como entradas. É nessa etapa que o agente analisa a arquitetura atual, procura alternativas e decide a melhor forma de implementar aquilo que foi especificado.

Uma mesma spec pode ter várias implementações possíveis. Podemos criar um novo serviço ou estender um existente, processar algo de forma síncrona ou assíncrona, reutilizar uma abstração ou criar outra.

O PLAN serve justamente para tomar essas decisões antes que elas virem código.

### O que compõe um bom PLAN?

O PLAN (aquele famoso `/plan` (que existe no copilot / claude etc que agora é personalizado com nossas exigências)) começa analisando **alternativas técnicas**. O agente precisa entender o que já existe, considerar abordagens diferentes e escolher uma delas levando em conta simplicidade, performance, manutenção e consistência com a arquitetura.

Depois vem o **design da solução**.

Aqui podemos detalhar componentes, sequência de chamadas, persistência, eventos, integrações e dependências. É onde diagramas, fluxogramas e outros elementos de system design começam a fazer mais sentido.

A diferença fica mais clara quando colocamos lado a lado:

**SPEC:** "Quando X acontecer, o sistema deve produzir Y."

**PLAN:** "Para fazer isso neste codebase, vamos alterar A, criar B, chamar C e persistir D."

O PLAN também quebra a implementação em tarefas menores. Cada tarefa deve ser pequena o suficiente para ser executada e validada em um único passo de codificação.

Isso ajuda a chegar em **atomic commits**, reduz o contexto necessário para cada etapa e torna muito mais fácil identificar onde alguma coisa deu errado.

É também aqui que detalhes mais específicos de implementação entram: módulos e providers do framework (comportamento e melhores práticas da linguagem tbm), migrations, arquivos que serão criados ou alterados, sequência das mudanças e comandos de validação.

BDD e TDD também podem reaparecer nessa etapa, mas agora com um papel mais técnico. A SPEC define os comportamentos que precisam ser atendidos; o PLAN pode definir como esses comportamentos serão cobertos por testes unitários, integração e outras verificações.

No final, o PLAN transforma a intenção em um caminho concreto de implementação.

## CODE: executando o plano sem reinventar a feature

Depois da SPEC e do PLAN, chega a hora do `/code`.

Aqui a intenção muda de novo: o agente não está mais tentando entender o problema e também não deveria estar redesenhando a solução. Ele recebe um plano já definido e passa a atuar como executor técnico.

O ponto principal desse prompt é reduzir improviso. Em vez de pedir algo genérico como "implemente essa feature", o agente recebe tarefas menores, contexto do projeto, critérios de aceitação e uma ordem de execução.

Também é aqui que o TDD começa a aparecer de forma mais concreta. Para cada parte do plano, o agente pode primeiro transformar o comportamento esperado em testes, depois escrever o mínimo necessário para fazê-los passar e, por fim, refatorar.

De forma simples:

> **Test → Implement → Refactor → Validate**

Outro ponto importante é deixar explícito que o agente deve **reutilizar o que já existe antes de criar novas abstrações**. Em codebases grandes, uma das formas mais rápidas de uma LLM aumentar a complexidade é resolver localmente um problema que o sistema já resolveu em outro lugar.

O `/code` também pode controlar o nível de autonomia. Mudanças pequenas podem seguir de forma contínua, enquanto alterações mais sensíveis podem ser executadas task por task, permitindo revisão humana entre as etapas.

No fim, a função dessa etapa é relativamente direta: **executar o PLAN respeitando os contratos definidos na SPEC e produzir código que já nasce acompanhado de mecanismos de verificação.**

Com isso, fechamos uma sequência bastante simples:

**SPEC → PLAN → CODE → VERIFY**

Cada etapa reduz um tipo diferente de incerteza. A SPEC reduz a incerteza sobre o que queremos, o PLAN sobre como vamos fazer e o CODE sobre como transformar essa decisão em uma implementação verificável.


## Juntando tudo

A partir daqui, o desenho começa a ficar mais interessante:

![sdd-llm](/images/fluxo-trabalho-sdd.png)

O ponto não é criar mais burocracia para desenvolver uma feature. É criar **memória e verificabilidade** para um processo que, sem isso, depende demais da conversa com a LLM.

No fim das contas, talvez o maior ganho do SDD não seja escrever mais documentação. É conseguir transformar uma intenção em algo que outro agente, outra pessoa ou até o próprio sistema consiga entender, executar e verificar.

E aí a gente volta para o começo: talvez não sejam necessários 18 agentes alucinando.

Talvez uma boa spec já resolva uma boa parte do problema.

## É isso...
É possível gerar features e sistemas com qualidade excepcional utilizando llm, vc só precisa estar no controle desde o inicio!

Vo deixar abaixo nas refs um repositório com exemplos de spec, plan e code para que vc possa utilizar nos seus projetos. Para cada ferramenta existe um padrão de uso, no copilot por exemplo, a gente pode criar qualquer markdown dentro de `projeto/.github/prompts/(spec/plan/code.prompt.md)`, enquanto no claude vc pode ter os mesmos arquivos como skills

> Deixar aqui um abraço ao senhor Zéé hudson que tive a honra de trabalhar junto, que um dia criou e difundiu o esqueleto que mencionei nesse artigo, desde lá minha visão mudou totalmente sobre o uso mais eficiente de llms :)

### Referências

- [Loop Engineering](https://addyosmani.com/blog/loop-engineering/)
- [Harness e loop engineering: por que importam mais que o modelo](https://www.youtube.com/watch?v=7paiK1n78mk)
- [Repositório com os arquivos mencionados](https://github.com/girorme/artigo-sdd-harness)