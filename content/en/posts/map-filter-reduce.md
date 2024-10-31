+++
author = "girorme"
title = "Clean and Elegant Code with Map, Filter, and Reduce"
date = "2024-10-31"
description = "Discover how map, filter, and reduce can transform your code: elegance, expressiveness, and conciseness in every line. Simplify your operations and write less while doing more!"
tags = [
    "functional",
    "programming",
    "php",
    "python",
    "javascript"
]
categories = [
    "functional",
    "programming",
    "php",
    "python",
    "javascript"
]
+++

Have you heard of `map`, `filter`, and `reduce`? If you're familiar with them, you’re definitely writing more expressive code. If not, let’s discover why you should use them and where this flexibility comes from...

![fn](/images/functional_programming_concept.png)

Most of us start programming in an imperative style: giving step-by-step instructions, telling the code what to do. You create loops, control variables, and write a ton of details to get to the final result. But did you know there’s a more elegant way to do this? With `map`, `filter`, and `reduce`, you can simplify the process. In this post, we’ll compare the traditional way with these functions, showing how they can make your code cleaner and more concise (examples in Python).

### 1. Example with `map`

Imagine you have a list of numbers and want to double the value of each one.

#### Imperative
In the imperative style, you’d have to create a new list, iterate over the numbers, and add each doubled value to the new list. Something like this:

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
doubled = []
for num in nums:
    doubled.append(num * 2)
{{< / highlight >}}

Here, you're telling the program **how** to do it: "Create an empty list, loop through it, double the numbers, and insert them one by one."

#### Functional with `map`
Now, using `map`, you express **what** you want: "Double each number in this list."

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
doubled = map(lambda x: x * 2, nums)
{{< / highlight >}}

Notice that you don’t need to mention anything about the empty list or how to traverse the items. `map` takes care of that. This makes the code **shorter** and **focused** on the transformation, not the process. *Less boilerplate, more results!*

### 2. Example with `filter`

Now, imagine you want to get only the even numbers from that list.

#### Imperative
Here, you’d need to do another loop, checking each number, and if it’s even, add it to a new list:

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
evens = []
for num in nums:
    if num % 2 == 0:
        evens.append(num)
{{< / highlight >}}

Once again, you’re telling the program **how** to proceed: "Check each number, see if it’s even, and put it in the list."

#### Functional with `filter`
With `filter`, it’s straightforward: "Select the evens."

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
evens = filter(lambda x: x % 2 == 0, nums)
{{< / highlight >}}

The magic here is that you focus only on the **filtering condition**, not on the control flow. This not only makes the code **cleaner** but also avoids common mistakes, like forgetting to add the item to the list or writing confusing logic in the loop.

### 3. Example with `reduce`

Let’s say you want to sum all the numbers in the list. In the imperative style, you’d have to declare an accumulator variable and iterate over the numbers, adding them one by one.

#### Imperative
Here’s the traditional approach:

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
total = 0
for num in nums:
    total += num
{{< / highlight >}}

Once again, you’re **telling how** to do it. And here, you already need an extra variable, `total`, to keep track.

#### Functional with `reduce`
With `reduce`, you express directly: "Reduce this list to a single value by summing the items."

{{< highlight python "linenos=table" >}}
from functools import reduce
nums = [1, 2, 3, 4]
total = reduce(lambda acc, x: acc + x, nums)
{{< / highlight >}}

Look how neat: everything you need is in a single line. `reduce` takes the list and accumulates the value along the sequence, and you don’t have to worry about additional variables or loop control.

### Why is using `map`, `filter`, and `reduce` elegant?

The main differentiator of functional functions compared to imperative style:

1. **Expressiveness**: Using these functions allows you to describe the **intent** of the code instead of detailing **step by step how** the operation should be done. The code becomes closer to the way we think about the problem, which improves **readability**.

2. **Abstraction of control flow**: You don’t have to worry about loops, indices, temporary variables, and all that junk that clutters the code with unnecessary details. This minimizes the chance of errors and improves **maintenance**.

3. **Easy composition**: The functional style allows you to **compose operations** intuitively. You can chain `map`, `filter`, and `reduce` without losing clarity:

   {{< highlight python "linenos=table" >}}
   nums = [1, 2, 3, 4]
   total = reduce(lambda acc, x: acc + x, filter(lambda x: x % 2 == 0, map(lambda x: x * 2, nums)))
   {{< / highlight >}}

   Here, you’re doubling the numbers, filtering the evens, and summing everything in a single expression. In imperative style, this would require multiple loops and variables, which would complicate the code.

4. **Immutability and less state**: These functions don’t modify the original data (unless you force it). This protects you from tricky debugging problems, like unexpected state changes, which are common in imperative style. In the functional world, data is sacred, and functions treat it without side effects.

### Other Languages

It’s important to highlight that you don’t need to use a functional language like Elixir to take advantage of functions like `map`, `filter`, and `reduce` (we used Python in the examples). These functions are available in various programming languages. Let’s see how to implement the same examples in PHP and JavaScript.

#### 1. `map`

**PHP:**

{{< highlight python "linenos=table" >}}
$nums = [1, 2, 3, 4];
$doubled = array_map(function($x) { return $x * 2; }, $nums);
{{< / highlight >}}

**JavaScript:**

{{< highlight javascript "linenos=table" >}}
const nums = [1, 2, 3, 4];
const doubled = nums.map(x => x * 2);
{{< / highlight >}}

#### 2. `filter`

**PHP:**

{{< highlight python "linenos=table" >}}
$nums = [1, 2, 3, 4];
$evens = array_filter($nums, function($x) { return $x % 2 === 0; });
{{< / highlight >}}

**JavaScript:**

{{< highlight javascript "linenos=table" >}}
const nums = [1, 2, 3, 4];
const evens = nums.filter(x => x % 2 === 0);
{{< / highlight >}}

#### 3. `reduce`

**PHP:**

{{< highlight python "linenos=table" >}}
$nums = [1, 2, 3, 4];
$total = array_reduce($nums, function($acc, $x) { return $acc + $x; }, 0);
{{< / highlight >}}

**JavaScript:**

{{< highlight javascript "linenos=table" >}}
const nums = [1, 2, 3, 4];
const total = nums.reduce((acc, x) => acc + x, 0);
{{< / highlight >}}

These examples show that, regardless of the language, the concept of higher-order functions (functions that take or return other functions) is widely applicable, allowing you to write more concise and expressive code.

### Summary: from imperative to elegant

When you see imperative code, you’re focused on **how** the operation should be done, which leads to more code and more implementation details. With `map`, `filter`, and `reduce`, you specify **what** you want to do, letting the language handle the details. This results in leaner, more readable code that’s less prone to errors.

These functions are like the perfect soundtrack for data manipulation: discreet, efficient, and they get the job done with style!