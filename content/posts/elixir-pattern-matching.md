+++
author = "girorme"
title = "Elixir no dia a dia - Pattern Matching"
date = "2022-03-15"
description = "Pattern matching é uma poderosa parte de Elixir que nos permite procurar padrões simples em valores, estruturas de dados, e até funções"
tags = [
    "elixir",
    "functional-programming"
]
categories = [
    "elixir",
    "programming"
]
series = ["Themes Guide"]
aliases = ["migrate-from-jekyl"]
+++

Pattern matching é uma poderosa parte de Elixir que nos permite procurar padrões simples em valores, estruturas de dados, e até funções

## Agenda
- [O Básico](#o-básico)
- [Extração de valores](#extração-de-valores)
- [Pattern matching em tudo!!!](#pattern-matching-em-tudo!!!)
- [head and tail](#head-and-tail)
- [Pin operator](#pin-operator)
- [Outras operações](#outras-operações)
- [Conclusão](#conclusão)
- [Ref](#ref)

![leafs](/images/leafs.jpg)

ref: https://elixirschool.com/pt/lessons/basics/pattern_matching

### O Básico
O operador `=` no elixir é tratado de forma difente comparado a outras linguagens. Chamamos de `match operator` esse carinha que além de extrair valores, pode até ser usado como "substituto" de estruturas de condições em alguns casos, além de armazenar valores.

Então tenha em mente que quando usamos o operador, estamos fazendo um match: se a operação do lado esquerdo dá match com a do lado direito, nós temos uma operação válida.

![tom-confuse](/images/tom-confuse.png)

pra descomplicar... ~~ou complicar mais~~

Isso quer dizer que, podemos fazer "comparações" e armazenar valores:

{{< highlight elixir "linenos=table" >}}
1 = 1
"rodrigo" = "rodrigo"

num = 1 
nome = "rodrigo"
{{< / highlight >}}

1 - As duas primeiras expressões são válidas pq literalmente: 1 é igual a 1 e a string "rodrigo" é igual a string "rodrigo"

2 - Nas expressões seguintes, o match operator atua como um "bindador" de valores, já que não temos um valor literal do lado esquerdo e sim uma varíavel, então o elixir sabe que deve associar o valor do lado direito ao esquerdo

### Extração de valores

Para fazer um paralelo, vale lembrar de funcionalidades de outras linguagem como o [Destructuring](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) do js ou até msm o [list](https://www.php.net/manual/en/function.list) do php (nas novas versões do php tbm temos um operador match: [match php](https://www.php.net/manual/pt_BR/control-structures.match.php)) para depois voltarmos ao pattern matching:

{{< highlight js "linenos=table" >}}
let arr = ["John", "Smith"]

// destructuring assignment
// sets firstName = arr[0]
// and surname = arr[1]
let [firstName, surname] = arr;

alert(firstName); // John
alert(surname);  // Smith
{{< / highlight >}}

List do php:

{{< highlight php "linenos=table" >}}
list($nome, $idade) = ["Rodrigo", 30];
echo $nome; // "Rodrigo"
echo $idade; // 30
{{< / highlight >}}

Por outro lado com o match operator podemos fazer extrações com qualquer tipo de dado da linguagem, vamos aos exemplos:

{{< highlight elixir "linenos=table" >}}
[first_n, second] = [1, 2] 

# first_n => 1
# second => 2

[1, second] = [1, 2] # podemos fazer match + extrações de forma bem simples

# second => 2

%{first_name: name, extra_info: info} = %{first_name: "foo", extra_info: "bar"}

# name => "foo"
# extra_info => "bar"
{{< / highlight >}}

Além de fazer extrações temos um cenário interessante no seguinte trecho:

{{< highlight elixir "linenos=table" >}}
[1, second] = [1, 2]
{{< / highlight >}}

Como estamos fazendo um match, a expressão é valida pq além do tipo do dado ser compatível (lista) temos o primeiro valor igual dos dois lados.

Podemos validar que os lados são comparados se passarmos valores diferentes, veja:

{{< highlight elixir "linenos=table" >}}
iex(1)> [1, second] = [1, 2]
[1, 2] # ok

iex(2)> [2, second] = [1, 2]
** (MatchError) no match of right hand side value: [1, 2]
{{< / highlight >}}

Na primeira expressão, ok, temos o número 1 do lado direito e do esquerdo. Na segunda temos um erro de match porque os valores não condizem dos dois lados.

Vale lembrar que o erro de match irá ocorrer literalmente em qualquer falha de comparação:

{{< highlight elixir "linenos=table" >}}
iex(1)> %{name: "Rodrigo", year: 2022} = %{name: "Rodrigo", year: 2022}

%{name: "Rodrigo", year: 2022}
# ok,
# aqui estamos apenas comparando os valores

iex(2)> %{name: name, year: 2022} = %{name: "Rodrigo", year: 2022}
# ok, estamos armazendo a string "Rodrigo" dentro da var name
# name => "Rodrigo"

iex(3)> %{name: name, year: 2021} = %{name: "Rodrigo", year: 2022}
# erro
** (MatchError) no match of right hand side value: %{name: "Rodrigo", year: 2022}
{{< / highlight >}}

na segunda expressão, conseguimos armazenar o valor "Rodrigo" na variável `name` mas na terceira não é possível, já que do lado direito temos o ano 2022 e do lado esquerdo temos o valor 2021.

### Pattern matching em tudo!!!
Na medida que nos habituamos com a linguagem percebemos que patterning matching está presente em muitas operações, porque a legibilidade do código aumenta e deixamos nossas intenções mais claras:

#### Funções
{{< highlight elixir "linenos=table" >}}
defmodule UserUtils do
  def extract_name(%{name: name_var}) do
    IO.puts "Name: #{name_var}"
  end
end

# chamada da função:
user = %{name: "Rodrigo", year: 2022}
UserUtils.extract_name(user)

# resultado:
# Name: Rodrigo
{{< / highlight >}}

É válido observar e já introduzir um ponto importante: podemos fazer matches parciais em mapas. Veja que definimos um mapa com 2 chaves: `name` e `year` mas na função `extract_name` conseguimos dar match somente na chave name. Nós literalmente falamos: 

- função espere um mapa que tenha a chave name e guarde o valor dela na variável `name_var`.

Agora imagine que além de querer extrair essa variável, queremos também ter certeza que o ano é 2022. 

Poderíamos implementar a função da seguinte forma:

{{< highlight elixir "linenos=table" >}}
defmodule UserUtils do
  def extract_name(%{name: name_var, year: 2022}) do
    IO.puts "Name: #{name_var}"
  end
end

# chamada da função:
user = %{name: "Rodrigo", year: 2022}
UserUtils.extract_name(user)

# resultado:
# Name: Rodrigo
{{< / highlight >}}

agora passando um mapa que não casa com o padrão:

{{< highlight elixir "linenos=table" >}}
defmodule UserUtils do
  def extract_name(%{name: name_var, year: 2022}) do
    IO.puts "Name: #{name_var}"
  end
end

# chamada da função:
user = %{name: "Rodrigo", year: 2021} # 2021 aqui não casa com o esperado pela função
UserUtils.extract_name(user)

# resultado:
** (FunctionClauseError) no function clause matching in UserUtils.extract_name/1

The following arguments were given to UserUtils.extract_name/1:

    # 1
    %{name: "Rodrigo", year: 2021}

    #cell:2: UserUtils.extract_name/1
{{< / highlight >}}

Dito isso podemos ter um fallback pra qualquer outro valor que não seja o esperado no match, já que caso não exista essa implementação vamos sempre tomar o erro como no exemplo acima. No exemplo a seguir utilizamos outra funcionalidade da linguagem que é o `multi clause function` que nos permite junto com pattern matching redeclarar uma função com argumentos diferentes:

{{< highlight elixir "linenos=table" >}}
defmodule UserUtils do
  def extract_name(%{name: name_var, year: qualquer_outro_ano}) do
    IO.puts "Name: #{name_var}, ano: #{qualquer_outro_ano}"
  end

  def extract_name(%{name: name_var, year: 2022}) do
    IO.puts "Name: #{name_var}"
  end

  
end

# chamada da função:
user = %{name: "Rodrigo", year: 2021}
UserUtils.extract_name(user)

# resultado
Name: Rodrigo, ano: 2021
:ok
{{< / highlight >}}

> A ordem das funções aqui importa, então o primeiro match que tem um padrão específico irá ser executado. Caso queira testar mude a ordem das funções e verá que mesmo que exista um match específico sempre a primeira instrução será executado

Caso não tenha reparado de início, o `multi clause function` nos permite por exemplo remover um if desnecessário (um para ano 2022 e qualquer outra coisa para ano != 2022), nós literalmente definimos funções para cada situação. No mundo real é mto comum encontrar funções com essas características, ex:

{{< highlight elixir "linenos=table" >}}
  def find(html_tree, selector_as_string) when is_binary(selector_as_string) do
    selectors = get_selectors(selector_as_string)
    find_selectors(html_tree, selectors)
  end

  def find(html_tree, selectors) when is_list(selectors) do
    find_selectors(html_tree, selectors)
  end

  def find(html_tree, selector = %Selector{}) do
    find_selectors(html_tree, [selector])
  end
{{< / highlight >}}

O exemplo acima vem da lib [floki](https://github.com/philss/floki). Perceba que temos a função `find` declara para vários cenários: 

- Quando o segundo argumento é uma string
- Quando o segundo argumento é uma lista
- Ou quando o segundo argumento é um tipo específico

Para complementar a parte de funções, veja esse exemplo:

{{< highlight elixir "linenos=table" >}}
# https://womanonrails.com/elixir-pattern-matching
defmodule Math do
  def minus?(), do: "No number"
  def minus?(x), do: x < 0
  def minus?(x, 2), do: "Suprice #{x}!"
  def minus?(x, y), do: x < 0 && y < 0
end

Math.minus?
#=> "No number"
Math.minus?(1)
#=> false
Math.minus?(1, 2)
#=> "Suprice 1!"
Math.minus?(1, 3)
#=> false
Math.minus?(1, -3)
#=> false
Math.minus?(-1, -3)
#=> true
{{< / highlight >}}

### head and tail
Uma operação comum em listas e que aparece sempre em recursão é a utilização do head and tail, que basicamente é extrair o primeiro valor de uma lista (head) e ter o resto dela (tail), ex:

{{< highlight elixir "linenos=table" >}}
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
{{< / highlight >}}

O mesmo resultado pode ser obtido da seguinte forma:

{{< highlight elixir "linenos=table" >}}
iex> list = [1, 2, 3]
iex> hd(list)
1
iex> tl(list)
[2, 3]
{{< / highlight >}}

### Pin operator
Variáveis no elixir podem ser reatribuídas/atualizadas e caso você não queira que isso aconteça (comum em comparações e fluxos específicos baseados em decisões (case/cond/etc)), o pin operator (`^`) pode ser utilizado:

{{< highlight elixir "linenos=table" >}}
# https://elixir-lang.org/getting-started/pattern-matching.html
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
{{< / highlight >}}

na segunda expressão nós evitamos que a variável seja reatribuida, nós estamos expressando: "Essa expressão só é valida se o valor for igual ao atribuido anteriormente"

Em outras operações:

{{< highlight elixir "linenos=table" >}}
# https://elixir-lang.org/getting-started/pattern-matching.html
iex> x = 1
1
iex> [^x, 2, 3] = [1, 2, 3]
[1, 2, 3]
iex> {y, ^x} = {2, 1}
{2, 1}
iex> y
2
iex> {y, ^x} = {2, 2}
** (MatchError) no match of right hand side value: {2, 2}
{{< / highlight >}}

Na última expressão o erro aparece porque já definimos o x = 1 e queremos apenas uma comparação e não uma atribuição

### Outras operações
O pattern matching aparece em muitas operações, e sabendo como funciona é possível executar fluxos complexos ou simplificados de várias formas:

#### case
{{< highlight elixir "linenos=table" >}}
case {:ok, "Hello World"} do
  {:ok, result} -> result
  {:error} -> "Uh oh!"
  _ -> "Catch all"
end
{{< / highlight >}}

{{< highlight elixir "linenos=table" >}}
case {1, 2, 3} do
  {1, x, 3} when x > 0 ->
    "Will match"
  _ ->
    "Won't match"
end
{{< / highlight >}}

#### maps
{{< highlight elixir "linenos=table" >}}
> key = "hello"
"hello"

> %{^key => value} = %{"hello" => "world"}
%{"hello" => "world"}

> value
"world"
{{< / highlight >}}

#### funções / funções anônimas
{{< highlight elixir "linenos=table" >}}
> greeting = "Hello"
"Hello"

> greet = fn
  (^greeting, name) -> "Hi #{name}"
  (greeting, name) -> "#{greeting}, #{name}"
end

> greet.("Hello", "Sean")
"Hi Sean"

> greet.("Mornin'", "Sean")
"Mornin', Sean"

> greeting
"Hello"
{{< / highlight >}}

#### tuplas
{{< highlight elixir "linenos=table" >}}
> {:ok, value} = {:ok, "Successful!"}
{:ok, "Successful!"}

> value
"Successful!"
{{< / highlight >}}

### Conclusão
Quanto mais a vontade se sentir com pattern matching, mais seu código ficará legível / pragmático :)