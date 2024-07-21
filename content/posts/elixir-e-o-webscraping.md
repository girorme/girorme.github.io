+++
author = "Girorme"
title = "Elixir e o webscraping"
date = "2021-04-03"
description = "Webscraping nunca sai da moda! xD"
tags = [
    "programming",
    "elixir",
    "functional programming"
]
categories = [
    "programming",
    "elixir",
    "functional programming"
]
+++


Para quem está sendo introduzido ao assunto, o conceito em grandes linhas é basicamente fazer requisições para uma ou mais urls e trabalhar em cima do resultado, 
traduzindo o conteúdo obtido para estruturas da linguagem

### Agenda
- [O Go-to do webscraping](#o-go-to-do-webscraping)
- [Configurando o projeto](#configurando-o-projeto)
	- [Criação via mix](#criação-via-mix)
	- [Instalação libs](#instalação-libs)
- [Features](#features)
	- [Parse argumentos cli](#parse-argumentos-cli)
	- [Request](#request)
	- [Extração de links](#extração-de-links)
	- [Salvar dados](#salvar-dados)
- [Conclusão](#conclusão)
- [Créditos e referência](#créditos-e-referência)

## O Go-to do webscraping
Para quem está sendo introduzido ao assunto, o conceito em grandes linhas é basicamente fazer requisições para uma ou mais urls e trabalhar em cima do resultado traduzindo o conteúdo obtido para estruturas da linguagem, dessa forma fica fácil a manipulação de várias formas, permitindo envio das informações para outro serviço, armazenamento local entra outras coisas

![searching](/images/spider.jpg)

## Configurando o projeto

Nossa "stack" do projeto:

- Elixir
- Bibliotecas
	- Tesla -> Client http
	- Floki -> Parser html


Caso não tenha elixir instalado: [Install elixir](https://elixir-lang.org/install.html)

### Criação via mix

Via mix criamos a estrutura inicial do projeto que nos permite com facilidade: instalar libs, fazer testes, criar executáveis, entre outras coisas

```text
$ mix new erlscrap
```

Saída:
```text
* creating README.md
* creating .formatter.exs
* creating .gitignore
* creating mix.exs
* creating lib
* creating lib/elscrap.ex
* creating test
* creating test/test_helper.exs
* creating test/elscrap_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd elscrap
    mix test

Run "mix help" for more commands.
```
### Instalação libs

**tesla**
_Tesla is an HTTP client loosely based on Faraday. It embraces the concept of middleware when processing the request/response cycle_

A lib tesla é muito simples de usar e é muito extensível. Excelente para requisições simmples ou até mesmo clientes complexos com middlewares

**floki**
_Floki is a simple HTML parser that enables search for nodes using CSS selectors._

Desenvolvida pelo nosso querido [Philipe sampaio](https://twitter.com/philipsampaio) a biblioteca floki é um parser html simples e perfomático que permite a manipulação de elementos html via seletores css o que significa que fica muito simples obter dados dentro de páginas

---

Inicialmente iremos editar a função `deps` dentro do arquivo `mix.exs`:

```elixir
defp deps do
  [
    {:tesla, "~> 1.3.0"},
    {:floki, "~> 0.29.0"}
  ]
end
```

Dessa forma após executar o comando: `mix deps.get` as depêndencias serão instaladas.

### Features

Nossa ferramenta terá 4 funcionalidades básicas que são:

### Parse argumentos cli

Antes de parsear os argumentos vindo da linha de comando iremos configurar nossa aplicação para que seja possível geramos um binário atráves do [escript](https://elixirschool.com/pt/lessons/advanced/escripts/) que pode rodar em qualquer sistema que tenha Erlang instalado (Existem outras formas de distribuição de binário que quero explorar em outros posts).

1. Inicialmente iremos alterar a função `project` do nosso `mix.exs`:

```elixir
def project do
  [
    app: :elscrap,
    version: "0.1.0",
    elixir: "~> 1.11",
    start_permanent: Mix.env() == :prod,
    deps: deps(),
    escript: escript() # Adicionamos essa chamada à função escript/0
  ]
end
```
2. após isso implementamos a nova função `escript/0` no mesmo arquivo:

```elixir
defp escript do
  [
    main_module: Elscrap.Cli, # Nosso módulo principal
    path: "bin/elscrap" # Saída do nosso binário
  ]
end
```

Feito isso podemos criar nosso módulo principal: `lib/cli.ex`. Como já temos nosso ponto de entrada definido, podemos começar com a implementação da função `main/1` que recebe os argumentos passados por cli:

```elixir
defmodule Elscrap.Cli do
  def main (args \\ []) do
    IO.inspect(args) # debug
  end
end
```

Para fins de testes agora podemos criar um binário e testar se os argumentos estão sendo passados corretamente:

```text
$ mix escript.build
$ ./bin/elscrap --extract-links --url "https://github.com" --save

["--extract-links", "--url", "https://github.com", "--save"]
```

Esses são os argumentos que queremos ter ao final da nossa aplicação. Quando executamos podemos ver a lista que é gerada logo abaixo.

Para tirar proveito desses argumentos podemos utilizar a função `OptionParser.parse/2` que nos da diversas opções de entrada. Muito similar ao que está na documentação do escript iremos definir nossa flag `--extract-links` como um booleano para facilitar as operações que temos em mente

Dito isso nossa função `main/1` muda e implementamos a função `parse_args/1`:

```elixir
def main (args \\ []) do
  args
  |> parse_args
end

defp parse_args(args) do
  {opts, _value, _} =
  args
  |> OptionParser.parse(switches: [extract_links: :boolean]) # aqui definimos que o parametro --extract-links será um bool

  opts
end
```

:warning: offtopic

Nesse trecho temos algumas expressões que se você não tem familiaridade com elixir pode estranhar... (Existe muito artigo sobre mas irei escrever algo sobre em breve) a primeira coisa em evidência é o operador pipe (`|>`) ele simplesmente cria um fluxo de execução entre funções passando o resultado de uma função como argumento para a próxima chamada, podemos criar algo como:

```elixir
[1, 2, 3]
|> Enum.map
|> Enum.filter
|> Foo.bar
```

Fazendo com que o resultado de uma função seja passado para outra função (é sempre bom lembrar que em elixir todas as funções retornam sua última expressão) então acaba sendo algo natural, para explorar mais: [Operador pipe - Elixir school](https://elixirschool.com/pt/lessons/basics/pipe-operator/)

Outra coisa que fica em evidência é o uso de `pattern matching` na expressão a seguir:

```elixir
{opts, _value, _} = ...
```

Nesse caso utilizamos como uma "destruct expression" para facilitar o paralelo com outras linguagens mas é claro que o pattern matching é muito mais que isso e recomendo fortemente a doc caso ainda não conheça: [Pattern matching - Elixir scrhool](https://elixirschool.com/pt/lessons/basics/pattern-matching/).

### Request

Agora entra a parte da requisição para a url passada através do parâmetro `url`.

A nossa função `main/1` ganha uma chamada extra e entra em jogo as funções `scrap/1` e `request/1`:

```elixir
def main (args \\ []) do
  args
  |> parse_args
  |> scrap # nova chamada
end

defp scrap(opts) do
  url = opts[:url] || nil

  unless url do 
    IO.puts("Url required")
    System.halt(0)
  end

  if opts[:extract_links] do
    links = request(url)
  end
end

defp request(url) do
  IO.puts("Extracting urls from: #{url}\n")
  {:ok, response} = Tesla.get(url)
  response.body
end
```

Fazer uma chamada com a lib `Tesla` é de fato simples:

```elixir
{:ok, response} = Tesla.get(url)
response.body
```

Logo em seguida retornamos o body da resposta esperamos uma resposta válida através do átomo :ok. Vamos utilizar a função `IO.inspect/1` para garantir que o retorno realmente funciona:

```elixir
{:ok, response} = Tesla.get(url)
IO.inspect(response.body)
```

Teste
```text
$ mix escript.build
...
$ ./bin/elscrap --extract-links --url "https://github.com"

Extracting urls from: https://github.com

"\n\n\n\n\n<!DOCTYPE html>\n<html lang=\"en\" class=\"html-fluid\">\n  <head>\n    <meta charset=\"utf-8\">\n  <link rel=\"dns-prefetch\" href=\"https://github.githubassets.com\">\n  <link rel=\"dns-prefetch\" href=\"https://avatars0.githubusercontent.com\">\n
...
```

Podemos partir então para a extração de links

### Extração de links

A função `scrap` agora evolui:

```elixir
if opts[:extract_links] do
  links = request(url)
  |> extract_links

  IO.puts("#{Enum.join(links, "\n")}")
end
```

Estamos expressando que iremos fazer a requisição via a função `request/1` passaremos o resultado para `extract_links/1` que retorna uma lista, logo após isso printamos nossa lista quebrando linha

Implementação da função `extract_links/1`:

```elixir
defp extract_links(response_body) do
  {:ok, document} = Floki.parse_document(response_body)

  links = document
  |> Floki.find("a")
  |> Floki.attribute("href")
  |> Enum.filter(fn href -> String.trim(href) != "" end)
  |> Enum.filter(fn href -> String.starts_with?(href, "http") end)
  |> Enum.uniq

  links
end
```

Algumas coisas acontecem aqui:

```elixir
{:ok, document} = Floki.parse_document(response_body)
# aqui esperamos apenas a resposta de quando da certo o parsing -> :ok
```

```elixir
links = document # passagem da variavel obtida do parsing acima
  |> Floki.find("a") # busque as tags (a) dentro do html
  |> Floki.attribute("href") # com as tags encontradas queremos o que está dentro de href (link)
  |> Enum.filter(fn href -> String.trim(href) != "" end) # Filtro para cada elemento iterado e retorne somente as strings não vazias
  |> Enum.filter(fn href -> String.starts_with?(href, "http") end) # Filtro para cada elemento iterado e retorno somente as strings que comecem com http
  |> Enum.uniq # Retorna uma nova lista apenas com urls não repetidas
```

Atualmente nosso módulo se encontra da seguinte forma:

```elixir
defmodule Elscrap.Cli do
  def main (args \\ []) do
    args
    |> parse_args
    |> scrap
  end

  defp parse_args(args) do
    {opts, _value, _} =
    args
    |> OptionParser.parse(switches: [extract_links: :boolean])

    opts
  end

  defp scrap(opts) do
    url = opts[:url] || nil

    unless url do
      IO.puts("Url required")
      System.halt(0)
    end

    if opts[:extract_links] do
      links = request(url)
      |> extract_links

      IO.puts("#{Enum.join(links, "\n")}")
    end
  end

  defp request(url) do
    IO.puts("Extracting urls from: #{url}\n")
    {:ok, response} = Tesla.get(url)
    IO.inspect(response.body)
  end

  defp extract_links(response_body) do
    {:ok, document} = Floki.parse_document(response_body)

    links = document
    |> Floki.find("a")
    |> Floki.attribute("href")
    |> Enum.filter(fn href -> String.trim(href) != "" end)
    |> Enum.filter(fn href -> String.starts_with?(href, "http") end)
    |> Enum.uniq

    links
  end
end
```

Agora já podemos rebuildar nossa aplicação e voilá:

```text
$ mix escript.build
...
$ ./bin/elscrap --extract-links --url "https://github.com"                   

Extracting urls from: https://github.com

https://docs.github.com/articles/supported-browsers
https://github.com/
https://lab.github.com/
https://opensource.guide
https://github.com/events
https://github.community
https://education.github.com
https://stars.github.com
https://enterprise.github.com/contact
https://enterprise.github.com/contact?ref_page=/&ref_cta=Contact%20Sales&ref_loc=billboard%20launchpad
https://www.npmjs.com
https://apps.apple.com/app/github/id1477376905?ls=1
https://play.google.com/store/apps/details?id=com.github.android
https://desktop.github.com/
https://cli.github.com
https://docs.github.com/github/managing-security-vulnerabilities/configuring-dependabot-security-updates
https://docs.github.com/discussions
https://enterprise.github.com/contact?ref_page=/&ref_cta=Contact%20Sales&ref_loc=footer%20launchpad
https://resources.github.com
https://github.com/github/roadmap
https://docs.github.com
http://partner.github.com/
https://atom.io
http://electronjs.org
https://services.github.com/
https://githubstatus.com/
https://github.com/contact
https://github.com/about
https://github.blog
https://socialimpact.github.com/
https://shop.github.com
https://twitter.com/github
https://www.facebook.com/GitHub
https://www.youtube.com/github
https://www.linkedin.com/company/github
https://github.com/github
```

Dessa forma obtemos todas as urls presentes na página do github.

### Salvar dados

Da forma que a ferramenta se encontra é fácil salvar o resultado para um arquivo usando o comando: `./bin/elscrap .. >> resultado.txt` mas para explorar um pouco mais podemos implementar uma função simples que cria um arquivo output quando quisermos

Dentro da nossa função `scrap/1` podemos adicionar um `if` checando se o parâmetro `--save` foi informado:

```elixir
if opts[:extract_links] do
  links = request(url)
  |> extract_links

  IO.puts("#{Enum.join(links, "\n")}")

  if opts[:save], do: save_links(url, links)
end
```

E agora a implementaçã da função `save_links/2`:

```elixir
defp save_links(url_id, links) do
  IO.puts("Saving links")

  file = "output/links.txt"

  content = links
  |> Enum.join("\n")

  case File.write(file, content) do
    :ok -> IO.puts("[#{url_id}] Links saved to: #{file}")
    {:error, reason} -> IO.puts("Error on save links: #{reason}")
  end
end
```

Criamos a pasta, rebuildamos a ferramenta e ao rodar o script agora com o novo parâmetro:

```text
./bin/elscrap --extract-links --url "https://github.com" --save            

Extracting urls from: https://github.com

...

Saving links
[https://github.com] Links saved to: output/links.txt
```
## Conclusão

Assim como outras linguagens é muito simples criar uma ferramenta de web scraping em elixir, temos a vantagem também de implementar funcionalidades assíncronas e perfomáticas de forma eficaz e simples além da expressividade da linguagem. É isso...

O repo da ferramenta está disponível no github: [Elscrap](https://github.com/girorme/elscrap)

## Créditos e referência

- [Elixir pattern matching](https://elixirschool.com/pt/lessons/basics/pattern-matching/)
- [Pipe operator](https://elixirschool.com/pt/lessons/basics/pipe-operator/)
- [Lib Floki](https://github.com/philss/floki)
- [Lib Tesla](https://github.com/teamon/tesla)