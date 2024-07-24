+++
author = "Girorme"
title = "Creating Elixir App Binaries Using Bakeware"
date = "2021-01-25"
description = "Bakeware is one of the alternatives to generate distributable binaries in the Elixir ecosystem"
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

As the king once said:

> Just generate a binary and it’ll be great.

![cat-box](/images/cat-box.jpg)

Just like in other languages like GOlang, with Bakeware we can compile an Elixir app into a single binary. Built on the excellent [releases](https://elixir-lang.org/getting-started/mix-otp/config-and-releases.html#releases) feature of Elixir in the latest versions, Bakeware even offers compression strategies using [Zstandard](https://en.wikipedia.org/wiki/Zstandard).

Inside the Bakeware repo, there are some [examples](https://github.com/bake-bake-bake/bakeware/tree/main/examples) of how to create binaries for different scenarios (CLI apps, Phoenix, etc.).

To demonstrate the tool, I will create a binary for an app I developed for banner grabbing called [binoculo](https://github.com/girorme/binoculo). Since we use the tool via CLI, distributing/releases a binary for this type of application becomes really interesting.

## Preparing the Ground
- You need to clone Bakeware outside the directory you want to compile.
- Next, some modifications are needed in the app's `mix.exs` file:

```elixir
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
```

- In the `application` function, we add the mod key to pass user arguments to the `main` function of the `Binoculo.CLI` module.

- In the `release` function, we define the app name via the first key in the keyword list:
  - overwrite: to replace a pre-existing binary
  - steps: Compilation steps using the `assemble/1` function from the Bakeware module
  - strip_beams: removes debug info related to beam

- And in deps, we basically include the library specifying the directory. The `runtime: false` option removes compile-time dependencies, so it's important to set it to false as recommended by the documentation.

Lastly, in the actual module, which in this case is `Binoculo.CLI`, we need to change the `main` function to receive command-line arguments:

```elixir
defmodule Binoculo.CLI do
  use Bakeware.Script

  ...

  @impl Bakeware.Script
  def main(args) do
    IO.puts("Binoculo CLI\n")
    args |> parse_args

    0
  end
```

This way, the arguments can still be parsed as they were using escript.

After that, with the `mix release` command, we can generate the binary:

```text
girorme@DESKTOP-MN7HFPO ~/repositorios/binoculo (binary-release) $ mix release
==> elixir_make
Compiling 1 file (.ex)
Generated elixir_make app

...
```

Once done, just run the binary inside the `_build/prod/rel/bakeware` folder:

```text
girorme@DESKTOP-MN7HFPO ~/repositorios/binoculo/_build/prod/rel/bakeware (binary-release) $ ./binoculo -h
Binoculo CLI

Binoculo 1.0.0 - Usage:
--head - Send an HTTP HEAD to server
--help | -h - Show Binoculo usage
--ip - CIDR notation/ip_range -> 192.168.0.1/24|192.168.0.1..192.168.0.255
--verbose - Show refused/error connections
-p | --port - Port to scan
-r | --read - Search for a specific word in response
-t | --threads - Number of threads

Ex: bin/binoculo --ip "192.168.0.1/24" -p 8080 -t 45 --head
```

For more information and use cases: [Bakeware-repo](https://github.com/bake-bake-bake/bakeware)

[]'s