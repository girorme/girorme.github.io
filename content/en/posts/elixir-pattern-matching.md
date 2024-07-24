+++
author = "girorme"
title = "Elixir in Daily Life - Pattern Matching"
date = "2022-03-15"
description = "Pattern matching is a powerful part of Elixir that allows us to search for simple patterns in values, data structures, and even functions."
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

Pattern matching is a powerful part of Elixir that allows us to search for simple patterns in values, data structures, and even functions.

## Agenda
- [The Basics](#the-basics)
- [Value Extraction](#value-extraction)
- [Pattern Matching Everywhere!!!](#pattern-matching-everywhere!!!)
- [Head and Tail](#head-and-tail)
- [Pin Operator](#pin-operator)
- [Other Operations](#other-operations)
- [Conclusion](#conclusion)
- [References](#references)

![leafs](/images/leafs.jpg)

ref: https://elixirschool.com/pt/lessons/basics/pattern_matching

### The Basics
The `=` operator in Elixir is treated differently compared to other languages. We call it the `match operator`, which not only extracts values but can also be used as a "substitute" for condition structures in some cases, in addition to storing values.

So keep in mind that when we use this operator, we are performing a match: if the operation on the left side matches the one on the right side, we have a valid operation.

![tom-confuse](/images/tom-confuse.png)

To simplify... ~~or complicate more~~

This means that we can perform "comparisons" and store values:

```elixir
1 = 1
"rodrigo" = "rodrigo"

num = 1 
nome = "rodrigo"
```

1 - The first two expressions are valid because literally: 1 is equal to 1, and the string "rodrigo" is equal to the string "rodrigo".

2 - In the subsequent expressions, the match operator acts as a "binder" of values since we don't have a literal value on the left side but rather a variable, so Elixir knows to associate the value on the right with the left.

### Value Extraction

To draw a parallel, consider functionalities in other languages such as [Destructuring](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) in JavaScript or even [list](https://www.php.net/manual/en/function.list) in PHP (in newer versions of PHP, there's also a match operator: [match php](https://www.php.net/manual/pt_BR/control-structures.match.php)) to return to pattern matching:

JavaScript Destructuring:

```js
let arr = ["John", "Smith"]

// destructuring assignment
// sets firstName = arr[0]
// and surname = arr[1]
let [firstName, surname] = arr;

alert(firstName); // John
alert(surname);  // Smith
```

PHP List:

```php
list($nome, $idade) = ["Rodrigo", 30];
echo $nome; // "Rodrigo"
echo $idade; // 30
```

On the other hand, with the match operator, we can perform extractions with any type of data in the language, here are some examples:

```elixir
[first_n, second] = [1, 2] 

# first_n => 1
# second => 2

[1, second] = [1, 2] # we can do match + extraction very simply

# second => 2

%{first_name: name, extra_info: info} = %{first_name: "foo", extra_info: "bar"}

# name => "foo"
# extra_info => "bar"
```

Besides extraction, there is an interesting scenario in the following snippet:

```elixir
[1, second] = [1, 2]
```

Since we are doing a match, the expression is valid because both sides are compatible (list) and the first value is equal on both sides.

We can validate that the sides are compared if we pass different values, see:

```elixir
iex(1)> [1, second] = [1, 2]
[1, 2] # ok

iex(2)> [2, second] = [1, 2]
** (MatchError) no match of right hand side value: [1, 2]
```

In the first expression, it's ok because we have the number 1 on both the right and left sides. In the second, there is a match error because the values on both sides do not match.

It is worth noting that the match error will occur literally for any comparison failure:

```elixir
iex(1)> %{name: "Rodrigo", year: 2022} = %{name: "Rodrigo", year: 2022}

%{name: "Rodrigo", year: 2022}
# ok,
# here we are just comparing values

iex(2)> %{name: name, year: 2022} = %{name: "Rodrigo", year: 2022}
# ok, we are storing the string "Rodrigo" in the var name
# name => "Rodrigo"

iex(3)> %{name: name, year: 2021} = %{name: "Rodrigo", year: 2022}
# error
** (MatchError) no match of right hand side value: %{name: "Rodrigo", year: 2022}
```

In the second expression, we managed to store the value "Rodrigo" in the variable `name`, but in the third, it is not possible since on the right side we have the year 2022 and on the left side, we have the value 2021.

### Pattern Matching Everywhere!!!
As we become more familiar with the language, we notice that pattern matching is present in many operations because it increases code readability and clarifies our intentions:

#### Functions
```elixir
defmodule UserUtils do
  def extract_name(%{name: name_var}) do
    IO.puts "Name: #{name_var}"
  end
end

# function call:
user = %{name: "Rodrigo", year: 2022}
UserUtils.extract_name(user)

# result:
# Name: Rodrigo
```

It is worth noting and introducing an important point: we can perform partial matches on maps. Note that we defined a map with 2 keys: `name` and `year`, but in the function `extract_name`, we were able to match only the `name` key. We literally said: 

- Function, expect a map that has the `name` key and store its value in the variable `name_var`.

Now imagine that besides wanting to extract this variable, we also want to ensure that the year is 2022. 

We could implement the function as follows:

```elixir
defmodule UserUtils do
  def extract_name(%{name: name_var, year: 2022}) do
    IO.puts "Name: #{name_var}"
  end
end

# function call:
user = %{name: "Rodrigo", year: 2022}
UserUtils.extract_name(user)

# result:
# Name: Rodrigo
```

Now passing a map that does not match the pattern:

```elixir
defmodule UserUtils do
  def extract_name(%{name: name_var, year: 2022}) do
    IO.puts "Name: #{name_var}"
  end
end

# function call:
user = %{name: "Rodrigo", year: 2021} # 2021 here does not match what the function expects
UserUtils.extract_name(user)

# result:
** (FunctionClauseError) no function clause matching in UserUtils.extract_name/1

The following arguments were given to UserUtils.extract_name/1:

    # 1
    %{name: "Rodrigo", year: 2021}

    #cell:2: UserUtils.extract_name/1
```

That said, we can have a fallback for any other value that is not expected in the match, as if this implementation does not exist, we will always encounter the error as shown above. In the following example, we use another language feature called `multi clause function`, which allows us to redeclare a function with different arguments:

```elixir
defmodule UserUtils do
  def extract_name(%{name: name_var, year: qualquer_outro_ano}) do
    IO.puts "Name: #{name_var}, year: #{qualquer_outro_ano}"
  end

  def extract_name(%{name: name_var, year: 2022}) do
    IO.puts "Name: #{name_var}"
  end
end

# function call:
user = %{name: "Rodrigo", year: 2021}
UserUtils.extract_name(user)

# result
Name: Rodrigo, year: 2021
:ok
```

> The order of functions here matters, so the first match with a specific pattern will be executed. If you want to test, change the order of the functions, and you will see that even if there is a specific match, the first instruction will always be executed.

If you didn't notice at first, the `multi clause function` allows us, for example, to remove an unnecessary `if` (one for the year 2022 and another for the year != 202

2); we literally define functions for each situation. In the real world, it's very common to find functions with these characteristics, e.g.:

```elixir
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
```

The example above comes from the [floki](https://github.com/philss/floki) library. Notice that we have the `find` function declared for various scenarios: 

- When the second argument is a string
- When the second argument is a list
- Or when the second argument is a specific type

To complement the functions part, see this example:

```elixir
# https://womanonrails.com/elixir-pattern-matching
defmodule Math do
  def minus?(), do: "No number"
  def minus?(x), do: x < 0
  def minus?(x, 2), do: "Surprise #{x}!"
  def minus?(x, y), do: x < 0 && y < 0
end

Math.minus?
#=> "No number"
Math.minus?(1)
#=> false
Math.minus?(1, 2)
#=> "Surprise 1!"
Math.minus?(1, 3)
#=> false
Math.minus?(1, -3)
#=> false
Math.minus?(-1, -3)
#=> true
```

### Head and Tail
A common operation in lists and that always appears in recursion is the use of head and tail, which basically extracts the first value of a list (head) and has the rest of it (tail), e.g.:

```elixir
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```

The same result can be obtained as follows:

```elixir
iex> list = [1, 2, 3]
iex> hd(list)
1
iex> tl(list)
[2, 3]
```

### Pin Operator
Variables in Elixir can be reassigned/updated, and if you do not want this to happen (common in comparisons and specific flows based on decisions (case/cond/etc)), the pin operator (`^`) can be used:

```elixir
# https://elixir-lang.org/getting-started/pattern-matching.html
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
```

In the second expression, we avoid the variable from being reassigned; we are expressing: "This expression is only valid if the value is equal to the previously assigned value."

In other operations:

```elixir
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
```

In the last expression, the error appears because we have already defined `x = 1` and we want only a comparison and not an assignment.

### Other Operations
Pattern matching appears in many operations, and knowing how it works, it's possible to perform complex or simplified flows in various ways:

#### Case
```elixir
case {:ok, "Hello World"} do
  {:ok, result} -> result
  {:error} -> "Uh oh!"
  _ -> "Catch all"
end
```

```elixir
case {1, 2, 3} do
  {1, x, 3} when x > 0 ->
    "Will match"
  _ ->
    "Won't match"
end
```

#### Maps
```elixir
> key = "hello"
"hello"

> %{^key => value} = %{"hello" => "world"}
%{"hello" => "world"}

> value
"world"
```

#### Functions / Anonymous Functions
```elixir
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
```

#### Tuples
```elixir
> {:ok, value} = {:ok, "Successful!"}
{:ok, "Successful!"}

> value
"Successful!"
```

### Conclusion
The more comfortable you become with pattern matching, the more readable and pragmatic your code will be :)