+++
author = "girorme"
title = "Elixir and Web Scraping"
date = "2021-04-03"
description = "Web scraping never goes out of fashion! xD"
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

For those being introduced to the topic, the concept in broad terms is basically making requests to one or more URLs and working on the result, translating the obtained content into the language's structures.

### Agenda
- [The Go-to for Web Scraping](#the-go-to-for-web-scraping)
- [Setting Up the Project](#setting-up-the-project)
    - [Creating via Mix](#creating-via-mix)
    - [Library Installation](#library-installation)
- [Features](#features)
    - [Parse CLI Arguments](#parse-cli-arguments)
    - [Request](#request)
    - [Extracting Links](#extracting-links)
    - [Save Data](#save-data)
- [Conclusion](#conclusion)
- [Credits and References](#credits-and-references)

## The Go-to for Web Scraping
For those being introduced to the topic, the concept in broad terms is basically making requests to one or more URLs and working on the result, translating the obtained content into the language's structures. This makes it easy to manipulate in various ways, allowing for information to be sent to another service, stored locally, among other things.

![searching](/images/spider.jpg)

## Setting Up the Project

Our project "stack":

- Elixir
- Libraries
    - Tesla -> HTTP Client
    - Floki -> HTML Parser

If Elixir is not installed: [Install Elixir](https://elixir-lang.org/install.html)

### Creating via Mix

Using Mix, we create the initial project structure which allows us to easily: install libraries, run tests, create executables, among other things.

```text
$ mix new erlscrap
```

Output:
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
### Library Installation

**tesla**
_Tesla is an HTTP client loosely based on Faraday. It embraces the concept of middleware when processing the request/response cycle._

The Tesla library is very simple to use and highly extensible. Excellent for simple requests or even complex clients with middlewares.

**floki**
_Floki is a simple HTML parser that enables search for nodes using CSS selectors._

Developed by our dear [Philipe Sampaio](https://twitter.com/philipsampaio), the Floki library is a simple and performant HTML parser that allows manipulation of HTML elements via CSS selectors, making it very easy to retrieve data from pages.

---

Initially, we will edit the `deps` function inside the `mix.exs` file:

```elixir
defp deps do
  [
    {:tesla, "~> 1.3.0"},
    {:floki, "~> 0.29.0"}
  ]
end
```

After running the command: `mix deps.get`, the dependencies will be installed.

### Features

Our tool will have 4 basic functionalities:

### Parse CLI Arguments

Before parsing arguments from the command line, we will configure our application so that it is possible to generate a binary using [escript](https://elixirschool.com/pt/lessons/advanced/escripts/) that can run on any system with Erlang installed (There are other binary distribution methods I want to explore in future posts).

1. First, we will modify the `project` function in our `mix.exs`:

```elixir
def project do
  [
    app: :elscrap,
    version: "0.1.0",
    elixir: "~> 1.11",
    start_permanent: Mix.env() == :prod,
    deps: deps(),
    escript: escript() # Added this call to the escript/0 function
  ]
end
```

2. Next, implement the new `escript/0` function in the same file:

```elixir
defp escript do
  [
    main_module: Elscrap.Cli, # Our main module
    path: "bin/elscrap" # Output path for our binary
  ]
end
```

With this done, we can create our main module: `lib/cli.ex`. As we already have our entry point defined, we can start implementing the `main/1` function which receives the arguments passed via CLI:

```elixir
defmodule Elscrap.Cli do
  def main (args \\ []) do
    IO.inspect(args) # debug
  end
end
```

For testing purposes, we can now create a binary and check if the arguments are passed correctly:

```text
$ mix escript.build
$ ./bin/elscrap --extract-links --url "https://github.com" --save

["--extract-links", "--url", "https://github.com", "--save"]
```

These are the arguments we want to have at the end of our application. When executed, we can see the list generated below.

To take advantage of these arguments, we can use the `OptionParser.parse/2` function which gives us various input options. Similar to what's in the escript documentation, we will define our `--extract-links` flag as a boolean to simplify the operations we have in mind.

Thus, our `main/1` function changes, and we implement the `parse_args/1` function:

```elixir
def main (args \\ []) do
  args
  |> parse_args
end

defp parse_args(args) do
  {opts, _value, _} =
  args
  |> OptionParser.parse(switches: [extract_links: :boolean]) # Here we define the --extract-links parameter as a boolean

  opts
end
```

:warning: off-topic

In this snippet, there are some expressions that you might find unusual if you are not familiar with Elixir... (There are many articles about this, but I will write something about it soon) The first thing to highlight is the pipe operator (`|>`), which simply creates an execution flow between functions, passing the result of one function as an argument to the next call. We can create something like:

```elixir
[1, 2, 3]
|> Enum.map
|> Enum.filter
|> Foo.bar
```

This makes the result of one function passed to another function (It is always good to remember that in Elixir all functions return their last expression). To explore more: [Pipe Operator - Elixir School](https://elixirschool.com/pt/lessons/basics/pipe-operator/)

Another thing highlighted is the use of `pattern matching` in the following expression:

```elixir
{opts, _value, _} = ...
```

Here, we use it as a "destructuring expression" to simplify the comparison with other languages, but pattern matching is much more than that and I strongly recommend the doc if you are not familiar with it: [Pattern Matching - Elixir School](https://elixirschool.com/pt/lessons/basics/pattern-matching/).

### Request

Now we move on to the request to the URL passed through the `url` parameter.

Our `main/1` function gains an extra call and introduces the `scrap/1` and `request/1` functions:

```elixir
def main (args \\ []) do
  args
  |> parse_args
  |> scrap # new call
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

Making a request with the Tesla library is indeed simple:

```elixir
{:ok, response} = Tesla.get(url)
response.body
```

Next, we return the body of the response and expect a valid response through the atom `:ok`. We will use the `IO.inspect/1` function to ensure that the return actually works:

```elixir
{:ok, response} = Tesla.get(url)
IO.inspect(response.body)
```

Test:
```text
$ mix escript.build
...
$ ./bin/elscrap --extract-links --url "https://github.com"

Extracting urls from: https://github.com

"\n\n\n\n\n<!DOCTYPE html>\n<html lang=\"en\" class=\"html-fluid\">\n  <head>\n    <meta charset=\"utf-8\">\n  <link rel=\"dns-prefetch\" href=\"https://github.githubassets.com\">\n  <link rel=\"dns-prefetch\" href=\"https://avatars0.githubusercontent.com\">\n
...
```

We can then move on to extracting links.

### Extracting Links



The `scrap` function now evolves:

```elixir
if opts[:extract_links] do
  links = request(url)
  |> extract_links

  IO.puts("#{Enum.join(links, "\n")}")
end
```

We are stating that we will make the request via the `request/1` function, pass the result to `extract_links/1` which returns a list, and then print our list line by line.

Implementation of the `extract_links/1` function:

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

Some things happen here:

```elixir
{:ok, document} = Floki.parse_document(response_body)
# Here we only expect a successful parsing response -> :ok
```

```elixir
links = document # passing the variable obtained from parsing above
  |> Floki.find("a") # search for <a> tags within the HTML
  |> Floki.attribute("href") # with the found tags, we want what's inside href (link)
  |> Enum.filter(fn href -> String.trim(href) != "" end) # Filter for each element iterated and return only non-empty strings
  |> Enum.filter(fn href -> String.starts_with?(href, "http") end) # Filter for each element iterated and return only strings starting with http
  |> Enum.uniq # Return a new list with only non-repeated URLs
```

Currently, our module looks like this:

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

Now we can rebuild our application and voilà:

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

Thus, we obtain all the URLs present on the GitHub page.

### Save Data

As it stands, it's easy to save the result to a file using the command: `./bin/elscrap .. >> result.txt`, but to explore a bit more, we can implement a simple function that creates an output file when needed.

Within our `scrap/1` function, we can add an `if` checking if the `--save` parameter was provided:

```elixir
if opts[:extract_links] do
  links = request(url)
  |> extract_links

  IO.puts("#{Enum.join(links, "\n")}")

  if opts[:save], do: save_links(url, links)
end
```

And now the implementation of the `save_links/2` function:

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

We create the folder, rebuild the tool, and now when running the script with the new parameter:

```text
./bin/elscrap --extract-links --url "https://github.com" --save            

Extracting urls from: https://github.com

...

Saving links
[https://github.com] Links saved to: output/links.txt
```
## Conclusion

Just like in other languages, it's very simple to create a web scraping tool in Elixir. We also have the advantage of implementing asynchronous and performant functionalities effectively and simply, in addition to the expressiveness of the language. That's it...

The tool repo is available on GitHub: [Elscrap](https://github.com/girorme/elscrap)

## Credits and References

- [Elixir Pattern Matching](https://elixirschool.com/pt/lessons/basics/pattern-matching/)
- [Pipe Operator](https://elixirschool.com/pt/lessons/basics/pipe-operator/)
- [Floki Library](https://github.com/philss/floki)
- [Tesla Library](https://github.com/teamon/tesla)