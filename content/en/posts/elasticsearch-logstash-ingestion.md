+++
author = "girorme"
title = "Data Ingestion with Elasticsearch and Logstash"
date = "2019-08-01"
description = "Elasticsearch has proven to be an excellent solution for high-performance data indexing, and data ingestion is a very interesting topic that offers many possibilities."
tags = [
    "elasticsearch",
    "data",
    "logstash"
]
categories = [
    "elasticsearch",
    "data",
    "logstash"
]
+++

> Over time, Elasticsearch has proven to be an excellent solution for high-performance data indexing, and data ingestion is a very interesting topic that offers many possibilities.

In this article, we will explore a bit of this power by creating a pipeline that consumes emails via IMAP. The goal is to demonstrate the ease and some subsequent examples of search, aggregations, and filters.

### Agenda
- [Who is Elasticsearch?](#who-is-elasticsearch?)
- [Who is Logstash?](#who-is-logstash?)
- [Setup](#setup)
- [Creating a Pipeline](#creating-a-pipeline)
- [Data Ingestion](#data-ingestion)
- [Executing Searches](#executing-searches)
- [A Taste of Filters and Aggregations](#a-taste-of-filters-and-aggregations)
- [That's It](#thats-it)
- [References](#references)

![searching](/images/searching.jpg)

Before getting hands-on, a brief introduction to the protagonists:

### Who is Elasticsearch?
> Elasticsearch provides near real-time search and analysis for all types of data. Whether you have structured or unstructured text, numeric data, or geospatial data, Elasticsearch can efficiently store and index them in a way that supports fast searches. You can go far beyond simple data retrieval and aggregate information to uncover trends and patterns in your data. As your data volume and queries grow, the distributed nature of Elasticsearch allows your deployment to scale seamlessly with it.

source: [Elastic](https://www.elastic.co/)

### Who is Logstash?
> Logstash is an open-source data collection engine with real-time pipelining capabilities. Logstash can dynamically unify data from disparate sources and normalize data to your chosen destinations. Clean and democratize all your data for various visualization and advanced downstream analytics use cases. Although Logstash originally drove innovation in log collection, its capabilities extend far beyond this use case. Any type of event can be enriched and transformed with a wide range of input, filter, and output plugins, with many native codecs further simplifying the ingestion process. Logstash accelerates your insights by leveraging a greater volume and variety of data.

source: [Elastic](https://www.elastic.co/)

### Setup
To simplify execution, we will use a Docker Compose setup that deploys an Elasticsearch + Logstash instance. It uses some configuration files. I created a repository if you want to test it: [GitHub - elastic-logstash-ingestion](https://github.com/girorme/elastic-logstash-ingestion)

### Creating a Pipeline
To define how data enters or exits Logstash, we use pipelines, which are basically instructions for [input](https://www.elastic.co/guide/en/logstash/current/input-plugins.html), [filter](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html), and [output](https://www.elastic.co/guide/en/logstash/current/output-plugins.html). As mentioned in the introduction, there are various ways to ingest and output data. For the sake of this article, we will use the [IMAP](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-imap.html) input plugin, which basically reads emails and indexes them in Elasticsearch.

To use/configure the plugin is simple. Here is an example of the `pipeline/pipeline.conf` file from the repository, configured with my credentials:

```text
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

In the above configuration file, we use an `input` configuration that specifies how Logstash obtains data and an `output` configuration where we use Elasticsearch. We define the host configured in Docker and set which index will be populated with the obtained email information.

### Data Ingestion

To finish, just run the command `docker-compose up`, which will consume by default the inbox (INBOX flag in IMAP) and populate the `gmail_inbox` index. After a few seconds of execution, you can already see that the index has been created:

```bash
$ curl -XGET http://localhost:9200/_cat/indices

green  open .monitoring-es-7-2021.08.02       XKIlhw_vQdGKsTgRv4ENmg 1 0   44 8   2.6mb   2.6mb
green  open .monitoring-es-7-2021.08.01       vAattcxgQWm4OekHsNNBbw 1 0 7146 0   2.4mb   2.4mb
yellow open logstash-2021.08.01-000001        80Qtu9I6SDiofQEI7gMOQA 1 1    0 0    208b    208b
green  open .monitoring-logstash-7-2021.08.01 R18bWi3SSPWym3vHAEchmA 1 0  139 0   235kb   235kb
---> yellow open gmail_inbox                       oNjxxicBR4WqZRwHSVpbEA 1 1   13 0 444.1kb 444.1kb
```

### Executing Searches

With the index created, we can perform searches in various ways (article coming soon).

Among the indexed fields, we will return only subject, from, and date, which were generated automatically:

```text
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
          "subject": "The main courses are on sale now!",
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
          "subject": "5 reasons to return to Guiabolso",
          "from": "Guiabolso <noreply@mailing.guiabolso.com.br>"
        }
      },
      
      ...
```

We can filter some keywords with a simple search if needed (in this case, all emails where the message field contains the word "news"):

request
```text
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
          "subject": "hey go to glory in the Stadium Series",
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
          "subject": "Your Monday night ride with Uber",
          "from": "Uber Receipts <uber.brasil@uber.com>"
        }
      }
    ]
  }
}
```

### A Taste of Filters and Aggregations

With the indexed data, we can do many cool things with Elasticsearch, such as using various filters and aggregations.

Below is an example of a `count_aggregation`, which counts how many emails we have from each "from" (note that we use `from.keyword`, which is a field generated by indexing of the keyword type used in sorting, aggregations, etc.), storing the result in the `senders_count` key:

request
```text
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
          "key": "\".NET SP Community (Meetup)\" <meetup-group-NSGNPppH-announce@meetup.com>",
          "doc_count": 8
        },
        {
          "key": "LinkedIn Job Alerts <jobalerts-noreply@linkedin.com>",
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
          "key": "Glassdoor Jobs <noreply@glassdoor.com>",
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

The result can become much more interesting by adding visualizations and charts in Kibana, which we can discuss in another article :) !!!

### That's It
This is a quick introduction that shows the power and flexibility of Logstash in conjunction with Elasticsearch. In the data ingestion example, we could have used a filter to get specific messages or with specific flags, also mixing with different outputs than Elasticsearch, which meets various needs.

I invite you to explore the amazing input, filter, and output plugins offered by Elastic:

- [Input plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)
- [Output plugins](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)
- [Filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)

### References
- https://www.elastic.co/guide/en/logstash/current/introduction.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html
- https://www.elastic.co/guide/en/logstash/current/docker.html