---
title: Elasticsearch - Ingestão de dados com logstash
date: 2021-08-01 19:28:15
tags:
- ES
- LOGSTASH
- IMAP
---

> Ao longo do tempo o elasticsearch tem se mostrado uma solução excelente para indexação de dados em alta perfomance e a ingestão de dados é um tema muito interessante que nos traz muitas possibilidades.
Nesse artigo iremos explorar um pouco desse poder criando um pipeline que consome emails via imap.
O objetivo é mostrar a facilidade e alguns exemplos posteriores de busca, agregações e filtros.

### Agenda
- [Quem é elasticsearch?](#quem-é-elasticsearch?)
- [Quem é logstash?](#quem-é-logstash?)
- [Setup](#setup)
- [Criando um pipeline](#criando-um-pipeline)
- [Ingestão de dados](#ingestão-de-dados)
- [Executando buscas](#executando-buscas)
- [Um gostinho de filtros e agregações](#um-gostinho-de-filtros-e-agregações)
- [É isso](#é-isso)
- [Referências](#Referências)

<div style="text-align:center"><img src="/images/searching.jpg" /></div>

Antes de qualquer mão na massa vale uma introdução aos protagonistas:

### Quem é elasticsearch?
> Elasticsearch fornece pesquisa e análise quase em tempo real para todos os tipos de dados. Quer você tenha texto estruturado ou não estruturado, dados numéricos ou dados geoespaciais, o Elasticsearch pode armazená-los e indexá-los com eficiência de uma forma que suporte pesquisas rápidas. Você pode ir muito além da simples recuperação de dados e agregar informações para descobrir tendências e padrões em seus dados. E à medida que seu volume de dados e consultas cresce, a natureza distribuída do Elasticsearch permite que sua implantação cresça perfeitamente junto com ela.

fonte: https://www.elastic.co/

### Quem é logstash?
> Logstash é um mecanismo de coleta de dados de código aberto com recursos de pipelining em tempo real. O Logstash pode unificar dinamicamente os dados de fontes distintas e normalizar os dados nos destinos de sua escolha. Limpe e democratize todos os seus dados para diversos casos de uso de visualização e análise downstream avançada. Embora o Logstash originalmente tenha impulsionado a inovação na coleta de logs, seus recursos se estendem muito além desse caso de uso. Qualquer tipo de evento pode ser enriquecido e transformado com uma ampla gama de plug-ins de entrada, filtro e saída, com muitos codecs nativos simplificando ainda mais o processo de ingestão. O Logstash acelera seus insights ao aproveitar um maior volume e variedade de dados.

fonte: https://www.elastic.co/

### Setup
Para facilitar a execução iremos utilizar um docker-compose que sobe uma estância de elasticsearch + logstash. Nele fazemos uso de alguns arquivos de configuração. Criei um repositório caso queira testar: https://github.com/girorme/elastic-logstash-ingestion


### Criando um pipeline
Para definirmos como os dados entram ou saem do logstash utilizamos pipelines, que são basicamente instruções de [entrada](https://www.elastic.co/guide/en/logstash/current/input-plugins.html), [filtro](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html) e [saida](https://www.elastic.co/guide/en/logstash/current/output-plugins.html). Como comentamos na introdução existem diversas formas de ingestão e output de dados, para exemplicar no artigo vamos utilizar o plugin de input de [imap](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-imap.html), que basicament lê emails e indexa no elasticsearch

Para usar/configurar o plugin é simples, aqui um exemplo do arquivo `pipeline/pipeline.conf` que está no repositório, no meu caso configurado com minhas credenciais:


```
input {
  imap {
    host => "imap.gmail.com"
    user => "mymail@gmail.com"
    password => "SECRET"
    port => 993
    secure => true
    strip_attachments => true
  }
}

output {
  elasticsearch {
    hosts => "elasticsearch:9200"
    index => "gmail_inbox"
    ecs_compatibility => disabled
  }
}
```

No arquivo de configuração acima estamos utilizando uma configuração de `input` que diz como o dado é obtido pelo logstash e no final uma configuração de `output` onde utilizamos elasticsearch. Defiminos o host que foi configurado no docker e setamos qual index será alimentado com as informações de email obtidas

### Ingestão de dados

Para finalizar, basta executar o comando `docker-compose up` que irá consumir por padrão a caixa de entrada (flag INBOX dentro do imap) e alimentar o índice gmail_inbox. Após alguns segundos de execução já é possível ver que o índice foi criado:

```
$ curl -XGET http://localhost:9200/_cat/indices

green  open .monitoring-es-7-2021.08.02       XKIlhw_vQdGKsTgRv4ENmg 1 0   44 8   2.6mb   2.6mb
green  open .monitoring-es-7-2021.08.01       vAattcxgQWm4OekHsNNBbw 1 0 7146 0   2.4mb   2.4mb
yellow open logstash-2021.08.01-000001        80Qtu9I6SDiofQEI7gMOQA 1 1    0 0    208b    208b
green  open .monitoring-logstash-7-2021.08.01 R18bWi3SSPWym3vHAEchmA 1 0  139 0   235kb   235kb
---> yellow open gmail_inbox                       oNjxxicBR4WqZRwHSVpbEA 1 1   13 0 444.1kb 444.1kb
```

### Executando buscas

Com o índice criado podemos executar buscas de várias formas (artigo em breve).

Entre os campos indexados iremos retornar apenas subject, from e date que foram gerados automaticamente:
```
GET http://localhost:9200/gmail_inbox/_search?pretty
content-type: application/json

{
    "_source": [
        "subject",
        "from",
        "date"
    ]
}
```

Response:

```json
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-encoding: gzip
content-length: 1470

{
  "took": 157,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 83,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "gmail_inbox",
        "_type": "_doc",
        "_id": "9LsOBXsBqoHsgYJoYeW4",
        "_score": 1.0,
        "_source": {
          "date": "Mon, 13 Jul 2020 23:26:53 -0700",
          "subject": "Os principais cursos estão em oferta agora!",
          "from": "Udemy <udemy@email.udemy.com>"
        }
      },
      {
        "_index": "gmail_inbox",
        "_type": "_doc",
        "_id": "w7sOBXsBqoHsgYJoJuXH",
        "_score": 1.0,
        "_source": {
          "date": "Sun, 12 Jul 2020 08:24:42 -0600",
          "subject": "5  motivos pra você voltar pro Guiabolso",
          "from": "Guiabolso <noreply@mailing.guiabolso.com.br>"
        }
      },
      
      ...
```

podemos filtrar algumas palavras chaves com busca simples caso seja necessário (_nesse caso todos os emails onde o campo message possuir a palavra news_):

request
```
GET http://localhost:9200/gmail_inbox/_search?pretty
content-type: application/json

{
    "query": {
        "match": {
            "message": {"query": "news"}
        }
    },
    "_source": [
        "subject",
        "from",
        "date"
    ]
}
```

response:
```json
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-encoding: gzip
content-length: 897

{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": 4.115555,
    "hits": [
      {
        "_index": "gmail_inbox",
        "_type": "_doc",
        "_id": "FLsOBXsBqoHsgYJokeaE",
        "_score": 4.115555,
        "_source": {
          "date": "Tue, 14 Jul 2020 23:28:35 +0000",
          "subject": "Introducing our first-ever newsletter",
          "from": "Codecademy <learn@codecademy.com>"
        }
      },
      {
        "_index": "gmail_inbox",
        "_type": "_doc",
        "_id": "EbsOBXsBqoHsgYJoi-a5",
        "_score": 3.6633658,
        "_source": {
          "date": "Tue, 14 Jul 2020 22:39:31 +0000",
          "subject": "How top businesses succeed in the work from home economy [Webinar]",
          "from": "IFTTT <mail@ifttt.com>"
        }
      },
      {
        "_index": "gmail_inbox",
        "_type": "_doc",
        "_id": "G7sOBXsBqoHsgYJonuap",
        "_score": 3.3366938,
        "_source": {
          "date": "Wed, 15 Jul 2020 11:17:20 +0000",
          "subject": "[PHP Classes] Weekly PHP Classes newsletter of Wednesday - 2020-07-15",
          "from": "PHP Classes Newsletter <list-newsletter@phpclasses.org>"
        }
      },
      {
        "_index": "gmail_inbox",
        "_type": "_doc",
        "_id": "xLsOBXsBqoHsgYJoKOXR",
        "_score": 2.7797134,
        "_source": {
          "date": "Sun, 12 Jul 2020 16:22:11 +0100",
          "subject": "girorme, vá para glória na Stadium Series ",
          "from": "PokerStars <events@starsaccount.com>"
        }
      },
      {
        "_index": "gmail_inbox",
        "_type": "_doc",
        "_id": "L7sOBXsBqoHsgYJosubh",
        "_score": 1.3502045,
        "_source": {
          "date": "Wed, 15 Jul 2020 14:25:24 +0000",
          "subject": "Elixir Radar 247",
          "from": "\"Hugo @ Elixir Radar\" <hugo@elixir-radar.com>"
        }
      },
      {
        "_index": "gmail_inbox",
        "_type": "_doc",
        "_id": "57sOBXsBqoHsgYJoWOUs",
        "_score": 0.60980225,
        "_source": {
          "date": "Mon, 13 Jul 2020 22:31:10 +0000",
          "subject": "Sua viagem de segunda-feira à noite com a Uber",
          "from": "Recibos da Uber <uber.brasil@uber.com>"
        }
      }
    ]
  }
}
```

### Um gostinho de filtros e agregações

Com os dados indexados podemos fazer muitas coisas legais com o elastic, como por exemplo utilizar filtros diversos e agregações.

Abaixo um exemplo de count_aggregation, trazendo quantos emails tenho de cada "from" (repare que utilizamos `from.keyword` que é um campo gerado pela indexação do tipo keyword que é utilizado em operações de sorting, agregações, etc), armezenando o resultado na chave `senders_count`:

request
```
GET http://localhost:9200/gmail_inbox/_search?pretty
content-type: application/json

{
    "size": 0,
    "_source": [
        "subject",
        "from",
        "date"
    ],
    "aggs" : {
    "senders_count": {
      "terms": {
        "field": "from.keyword"
      }
    }
  }
}
```

response:
```json
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-encoding: gzip
content-length: 560

{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 83,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "senders_count": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 49,
      "buckets": [
        {
          "key": "\"Comunidade .NET SP (Meetup)\" <meetup-group-NSGNPppH-announce@meetup.com>",
          "doc_count": 8
        },
        {
          "key": "Alertas de vaga do LinkedIn <jobalerts-noreply@linkedin.com>",
          "doc_count": 5
        },
        {
          "key": "\".NET São Paulo\" <info@meetup.com>",
          "doc_count": 3
        },
        {
          "key": "Medium Daily Digest <noreply@medium.com>",
          "doc_count": 3
        },
        {
          "key": "The Hacker News <news@news.nl00.net>",
          "doc_count": 3
        },
        {
          "key": "Todoist <daily-digest@todoist.com>",
          "doc_count": 3
        },
        {
          "key": "Vagas do Glassdoor <noreply@glassdoor.com>",
          "doc_count": 3
        },
        {
          "key": "\"stale[bot]\" <notifications@github.com>",
          "doc_count": 2
        },
        {
          "key": "GitHub <noreply@github.com>",
          "doc_count": 2
        },
        {
          "key": "Guiabolso <noreply@mailing.guiabolso.com.br>",
          "doc_count": 2
        }
      ]
    }
  }
}
```

O resultado pode ficar mpuito mais interessante juntando visualizações e gráficos no kibana que podemos discutir em outro artigo :) !!!

### É isso
Essa é uma introdução rápida que mostra o poder e flexibilidade do logstash em conjunto com elasticsearch. No exemplo de ingestão de dados poderíamos ter utilizado um filtro para pegar mensagens específicas ou com flags específicas também mesclando com saídas diferente que a do elasticsearch o que atende à varias necessidades

Te convido a explorar os plugins sensacionais de input, filter e output que são oferecidos pela elastic:

- [Input plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)
- [Output plugins](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)
- [Filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)

### Referências
- https://www.elastic.co/guide/en/logstash/current/introduction.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html 
- https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html
- https://www.elastic.co/guide/en/logstash/current/docker.html