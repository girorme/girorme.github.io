+++
author = "girorme"
title = "TDD Helps a Lot!"
date = "2024-08-19"
description = "TDD helps a lot, and I'll show you why!"
tags = [
    "tdd",
    "programming"
]
categories = [
    "tdd",
    "programming"
]
+++

TDD (Test-Driven Development) not only improves code quality but also encourages practices that keep development agile and anticipate problems, resulting in more robust and easier-to-maintain software.

![tdd-helps-a-lot-1](/images/tdd-ajuda-muito-1.jpg)

So, which side are you on? Love or hate tests? 😄

# Agenda

1. **Introduction to TDD**
   - What is Test-Driven Development (TDD)?

2. **One of TDD’s Many Benefits**

3. **TDD in Practice**
   - Example of usage and implementation

4. **Conclusion**
   - Final thoughts on adopting TDD

## Introduction to TDD

#### What is Test-Driven Development (TDD)?

> Without getting too philosophical: In TDD, you write the test before the actual implementation.

And if you find that strange or just plain annoying, hahaha, it’s probably because you haven’t yet reaped all the benefits this practice offers us as ~bug implementers~ devs.

As weird as it might seem, it works... and there’s a famous image to explain it, which is everywhere (I made my version on [excalidraw](https://excalidraw.com/)):

![tdd-helps-a-lot-2](/images/tdd-ajuda-muito-2.png)

Soon enough, we’ll dive into the methodology in practice here in the article. Even though it's cool, we won’t go into the history of TDD, but it’s worth understanding the motivations:

> https://en.wikipedia.org/wiki/Test-driven_development

In short, the technique stems from a simple reflection: If tests are so good and provide code quality, why not take it a step further and write the tests before the code itself?

## One of TDD’s Many Benefits

> The ability to express the implementation idea through the test. When we write a test before the code, we orchestrate the main idea. We test it the way we’d like to use the "API" of that code.

Even if you apply various patterns, sooner or later (often more sooner than later xD), you won’t write the code in its best form. It’s very common for these design "flaws" to be detected during the writing of a test.

You’ve probably been through this... when writing a test and trying to mock some response, you found it hard to make the behavior flexible and easy to test. Usually, the outcome of this difficulty is either refactoring the code (or simply creating a backlog task :p). You could greatly reduce this frustration by adopting TDD...

### TDD in Practice

By design, TDD forces us to create simple implementations so the test can pass. So, from the beginning, we’re refactoring the code.

What might seem like an extra learning curve (there is a natural curve, but once overcome, it brings incredible maturity to your codebase) is actually the story of your code being told.

#### Example of Usage and Implementation

Imagine the following requirement:

```
As a player, I want a simple, classic fighting game, So I can choose a character with X powers.
```

We could also have acceptance criteria like:

```
- When I select a character, I should choose their powers.
- After choosing the powers, I should start the game with the selected character.
```

There are many ways to implement the described features. Using TDD, we could simply start with the following test snippet (*The test could be much shorter, but it’s worth seeing this case where we "inject" the character into the game*):

*pseudo-oop-code:*

```python
import my_test_framework

fun test_choose_character(): 
    game = new Game()

    character = new Character('Ryu') 
    character->add_power('hadouken')

    game->add_character(character)

    assert game instanceof Game 
    assert character instanceof Character 
    assert character->get_powers() in ['hadouken'] 
    assert game->get_characters_count() == 1
...
```

We’ve created four test scenarios:

- We check if `new game()` is indeed a valid instance of `Game`.
- We check if `new character('Ryu')` is indeed a valid instance of `Character`.
- We check if our character has a power called `hadouken`.
- We check if we have 1 character inside the game after adding the character above.

> At this point, regardless of the language, if you run the test, it’ll fail. It’ll complain that `Game` and `Character` don’t exist and have never heard of the functions/methods `add_character`/`get_powers`/`get_characters_count`.

This is the red phase from the first image...

Right away, the main benefit I mentioned earlier: *We express how the code should behave.*

Even though the code doesn’t exist, you’re already expressing the idea.

With that said, you can now write the implementations in their simplest form, just to get the tests to pass. Imagine the numbers below are steps, and each time I rerun the tests, I’ll have more "mature" versions meeting the test’s needs step by step:

1.
```python
class Game: 
    fun add_character(character): 
        pass

    fun get_characters_count(): 
        return 1
```

*I run the test again, it partially passes, continues...*

2.
```python
class Character: 
    fun add_power(power): 
        pass

    fun get_powers(): 
        return ['hadouken']
```

*I run the test again, it partially passes, continues...*

Here, as simple as it is, the tests pass, and now you literally need to finish telling the story. You will implement it piece by piece, run the tests, and make it work exactly as planned!

Interesting, right? :)

### Quick Tips to Get Started

Here are some tips for starting with TDD:

- **Start with small and simple tests:** Write tests for the most basic and easy-to-implement parts of your code.
- **Test one functionality at a time:** Focus on a single function or behavior before moving on to the next.
- **Write tests that fail first:** A failing test confirms that the functionality hasn’t been implemented yet.
- **Refactor frequently:** After making the test pass, review the code to keep it clean and efficient.
- **Use quick feedback:** Run the tests constantly to get immediate feedback on your progress.
- **Don’t worry about writing "perfect code":** The code can be improved during refactoring; the important thing is to ensure it works correctly.
- **Be patient:** TDD may seem slow at first, but practice leads to efficiency.

### Conclusion

To wrap things up... it’s worth remembering that TDD is more than just a development technique; it’s a mindset shift. By adopting this approach, you’re not just writing tests; you’re shaping how your code evolves, with more confidence and fewer unpleasant surprises.

So, if you haven’t given TDD a try yet, now you know what to do...
Hope this helps!