+++
author = "Girorme"
title = "Autoloading in PHP Using Composer"
date = "2019-09-01"
description = "Learn how to create a project using Composer and the PSR-4 standard for autoloading"
tags = [
    "php",
    "composer"
]
categories = [
    "php",
    "composer",
    "programming"
]
aliases = ["migrate-from-jekyl"]
+++

We have PHP before and after Composer, and you need to know how to manage your project using this amazing tool :)

## TL;DR
> Learn how to create a project using Composer and the PSR-4 standard for autoloading

### It was all wild back then
You've surely seen or used the `require`/`require_once` or even `include` functions while programming with PHP.
These functions load the entire content of the requested file into memory, making it impractical to use them uncontrollably in medium/large scale projects.

Fortunately, over time, *[specifications have emerged](https://www.php-fig.org/)*, and the way we program in PHP (or should program... haha) has changed a lot. 
Standards have been adopted, giving us maturity/[interoperability](https://pt.wikipedia.org/wiki/Interoperabilidade) in development.

Among these specifications, we have some related to class autoloading (PSR-0 and PSR-4). PSR-0 is now deprecated, and as a replacement, we have ¹[PSR-4](https://www.php-fig.org/psr/psr-4/), which provides a standard for structuring our project to have efficient, performant, and on-demand autoloading (Files are only loaded into memory when we need them!).
For this, we have some well-defined rules that we will put into practice by creating a project using Composer ([I’ve briefly written about Composer if you're not familiar with it...](http://girorme.github.io/2017/07/23/gerenciando-bibliotecas-php-composer/)).

### Creating the project with Composer
Once you have Composer ready to use, we can create the project:

```bash
$ mkdir autoload-with-composer && cd autoload-with-composer
$ composer init
```

The `composer init` command asks us for some information about the project, like package name, version, license, etc.
I left it like this:

```text
Package name (<vendor>/<name>) [rodrigo/autoload-com-composer]: girorme/autoload-with-composer
Description []:
Author [girorme <rodrigo.girorme@gmail.com>, n to skip]:
Minimum Stability []:
Package Type (e.g. library, project, metapackage, composer-plugin) []: project
License []: MIT
Would you like to define your dependencies (require) interactively [yes]? no
Would you like to define your dev dependencies (require-dev) interactively [yes]? no
```

After that, the `composer.json` file is created, and we can start our experiment.
Now, we will define the folder/file structure that the specification expects.
The structure looks like this:

```text
- src/
  - App/
- composer.json
- index.php
```

Inside `src/`, we create the `App` folder, which will act as our namespace. This folder will be our "vendor namespace." So, if we create a file `Battle.php` inside `src/App`, we will call it using: `use App\Battle`. If we want a sub-namespace `Char`, we need to create a folder inside `app`: `src/App/Char`. And how do we call a class inside Char named `Ryu` with the file name `Ryu.php`, for example? Simple: `use App\Char\Ryu`.

Before that, for Composer to autoload the classes, we need to edit the `composer.json` file, mapping this namespace so that it does the heavy lifting. The file will look like this:

```json
{
    "name": "girorme/autoload-with-composer",
    "type": "project",
    "license": "MIT",
    "authors": [
        {
            "name": "girorme",
            "email": "rodrigo.girorme@gmail.com"
        }
    ],
    "require": {},
    "autoload": {
        "psr-4": {
            "App\\": "src/App"
        }
    }
}
```

Before diving into the code, it’s essential to know that the specification requires a well-defined structure, as mentioned, and we’ll demonstrate this in the next examples.

After modifying the `composer.json` file, it's necessary to run the `dump-autoload` command to recognize our structure:

```text
$ composer dump-autoload
```

Now we can create our classes and see autoloading in action. Let’s create some classes to represent Street Fighter characters and other classes for some attributes. First, let's see autoloading in action.
We will initially create a file called `Battle.php` inside `src/App`:

The `Battle.php` class:

```php
<?php

namespace App;

class Battle
{
    private $chars;

    public function __construct($fightTitle)
    {
        $this->fightTitle = $fightTitle;
    }

    public function addChar($char)
    {
        $this->chars[] = $char;
    }

    public function startBattle()
    {
        echo sprintf('%s Fight!' . PHP_EOL, $this->fightTitle);
    }
}
```

Remember that we defined our "vendor namespace" as `App`, so we need to reference it at the beginning of the file.

Now let's create our entry point inside the `index.php` file, making Composer bring in the requested files/classes:

```php
# index.php

<?php

require_once __DIR__ . '/vendor/autoload.php';

use App\Battle;

$battle = new Battle('Street Fighter!');
$battle->start();
```

After including just the `/vendor/autoload.php` file, which is generated by Composer, we will have all the requested classes on demand.

The output of the above code will be:

```bash
$ php index.php
Street Fighter! Fight!
```

Done! You already have autoloading working. Now we can put other points into practice, demonstrating how we can have subdirectories.

Let's add some characters and attributes to the battle. I’ll leave the final version of the `index.php` file here, but if you want to check out the source code of the example -> [composer-autoload-example](https://github.com/girorme/composer-autoload-example).

Our final structure will look like this:

```text
- src
  - App
    - Attributes
      - Hadouken.php
      - Shoryuken.php
    - Char
      - Ken.php
      - Ryu.php
  - Battle.php
- composer.json
- index.php
```

Our goal is basically:
> Create two fighters 
  > Add attributes to them 
  > Create a battle
  > Add the fighters to the battle
  > Display the fighters and attributes

Final version of the index.php file:

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

use App\Battle;
use App\Char\{Ryu, Ken};
use App\Attributes\{Hadouken, Shoryuken};

$hadouken = new Hadouken();
$ryu = new Ryu($hadouken);

$shoryuken = new Shoryuken();
$ken = new Ken($shoryuken);

$battle = new Battle('Street Fighter!');
$battle->addChar($ryu);
$battle->addChar($ken);
$battle->start();
```

Output:

```bash
$ php index.php
Street Fighter! Fight!
Char: Ryu, Power: Hadoooouken
Char: Ken, Power: Shooooooryuken
```

See how simple it is to use the classes:

```php
use App\Battle;
use App\Char\{Ryu, Ken};
use App\Attributes\{Hadouken, Shoryuken};
```

> ³ * **Note**: the notation `App\Char\{Ryu, Ken};` is a feature of PHP7 that simplifies the use of multiple classes. I could represent the same using:

```php
use App\Char\Ryu;
use App\Char\Ken;
```

Now, check out in the source code that we have both a class at the first level, which is the case of `App\Battle`, and sub-namespaces, which is the case of chars and attributes `App\Char\Ryu` and `App\Attributes\Hadouken`. Open these files, see how it was done, change the structure, and understand how PSR-4 works! :)

That's it, folks!

## References

- [1] [PSR-4](https://www.php-fig.org/psr/psr-4/)
- [2] [Composer](https://getcomposer.org/)
- [3] [PHP Group use declarations](https://www.php.net/manual/pt_BR/language.namespaces.importing.php#language.namespaces.importing.group)
