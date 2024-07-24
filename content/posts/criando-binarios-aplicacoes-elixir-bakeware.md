+++
author = "girorme"
title = "Criando binários de aplicações Elixir utilizando bakeware"
date = "2021-01-25"
description = "Bakeware é uma das alternativas para gerar binários distribuíveis no ecossistema elixir"
tags = [
    "elixir",
    "programming",
    "bakeware"
]
categories = [
    "elixir",
    "programming",
    "bakeware"
]
+++

Já dizia o rei:

> Gera um binário ai que fica legal

![cat-box](/images/cat-box.jpg)

Similar ao que temos em outras linguagens como GOlang, com o bakeware conseguimos compilar uma aplicação elixir em um único binário. Criado nos ombros da feature excelente de [releases](https://elixir-lang.org/getting-started/mix-otp/config-and-releases.html#releases) do elixir nas últimas versoes, o bakeware oferece até estratégias de compressão utilizando [Zstandard](https://en.wikipedia.org/wiki/Zstandard).

Dentro do repo do bakeware, existem alguns [exemplos](https://github.com/bake-bake-bake/bakeware/tree/main/examples) de como criar binários considerando alguns cenários (Cli apps, phoenix, etc)

Para demonstrar a ferramenta irei criar o binário de uma aplicação que desenvolvi para bannergrab chamada [binoculo](https://github.com/girorme/binoculo). Utilizamos a ferramenta via cli então a distribuição/releases de um binário para esse tipo de aplicação se torna muito interessante.


## Preparando o terreno
- É necessáro clonar o bakeware fora do diretório que queremos compilar
- Em seguida é necessário algumas modificações no arquivo `mix.exs` da aplicação:

{{< highlight elixir "linenos=table" >}}
def application do
  [
    ...,
    mod: {Binoculo.CLI, []}
  ]
end

defp release do
  [
    binoculo: [
      overwrite: true,
      steps: [:assemble, &Bakeware.assemble/1],
      strip_beams: Mix.env() == :prod
    ]
  ]
end

defp deps do
  [
    {:iplist, "~> 1.0.0"},
    {:bakeware, path: "../bakeware", runtime: false}
  ]
end
{{< / highlight >}}

- dentro da função `application` nós adicionamos a chave mod para  passar argumentos do usuário para a função `main` do módulo `Binoculo.CLI`

- dentro da função `release` definimos o nome da aplicação via a primeira chave da keywordlist
  - overwrite: para substituir um binário pré-existente
  - steps: Steps de compilação utilizando a função `assemble/1` do módulo Bakeware
  - strip_beams: remove info de debug relacionado à beam

- e em deps basicamente incluímos a biblioteca especificando o diretório. A opção `runtime: false` remove depedências de compilação então é importante ser false conforme recomenda a documentação

E por último dentro do módulo de fato que no caso é `Binoculo.CLI` devemos alterar a funçaõ `main` para receber os argumentos da linha de comando:

{{< highlight elixir "linenos=table" >}}
defmodule Binoculo.CLI do
  use Bakeware.Script

  ...

  @impl Bakeware.Script
  def main(args) do
    IO.puts("Binoculo cli\n")
    args |> parse_args

    0
  end
{{< / highlight >}}

dessa forma os argumentos podem continuar sendo parseados como era feito utilizando o escript.

após isso com o comando `mix release` é possível gerar o binário:

```text
girorme@DESKTOP-MN7HFPO ~/repositorios/binoculo (binary-release) $ mix release
==> elixir_make
Compiling 1 file (.ex)
Generated elixir_make app

...
```

Ao término basta executar o binário dentro da pasta `_build/prod/rel/bakeware`:

```text
girorme@DESKTOP-MN7HFPO ~/repositorios/binoculo/_build/prod/rel/bakeware (binary-release) $ ./binoculo -h
Binoculo cli


Binoculo 1.0.0 - Usage:
--head - Send a http HEAD to server
--help | -h - Show Binoculo usage
--ip - CIDR notation/ip_range -> 192.168.0.1/24|192.168.0.1..192.168.0.255
--verbose - Show refused / error connections
-p | --port - Port to scan
-r | --read - Search for especific word in response
-t | --threads - Number of threads

Ex: bin/binoculo --ip "192.168.0.1/24" -p 8080 -t 45 --head
```
Para mais informações e casos de uso: [Bakeware-repo](https://github.com/bake-bake-bake/bakeware)

[]'s