+++
author = "girorme"
title = "Managing Libraries in PHP with Composer"
date = "2017-07-23"
description = "Composer is indispensable for developing mature applications in PHP"
tags = [
    "php",
    "composer",
    "packagist"
]
categories = [
    "php",
    "composer",
    "packagist"
]
series = ["Themes Guide"]
aliases = ["migrate-from-jekyl"]
+++

Composer is a dependency manager for PHP. 
You can also use it for autoloading management, task scripting and other things


## Installing Composer

#### Linux
[Composer Linux](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos)

#### Windows
[Composer Windows](https://getcomposer.org/doc/00-intro.md#installation-windows)

If everything went well, you can run the command:

```text
$ composer
```

And several options will be displayed.

Where are the Composer libraries?
--

In the simplest way, when we manage libraries with Composer, we are fetching these libraries from [Packagist](https://packagist.org/).

Creating our Hello World with Composer
--

Create the folder:

```bash
$ mkdir hello-world-composer
```

Now let's initialize Composer inside our folder:

```bash
$ composer init
```

When running the above command, we will be asked about the author's name, project type, license, etc.

Fill in all the information as guided, and when it comes to the point where Composer asks if you want to define the project's libraries, type N so we can do it manually. You will also be asked if you want to set up require-dev, which are dependencies used for local development/testing, etc. We will understand this soon :)

At the end of the process of generating the `composer.json` file, it will look like this:

```json
{
    "name": "girorme/hello-world-composer",
    "description": "Composer test project",
    "type": "project",
    "license": "MIT",
    "authors": [
        {
            "name": "Rodrigo",
            "email": "girorme@gmail.com"
        }
    ],
    "require": {}
}
```

From this moment, we can already start playing around with Composer.

Installing Libraries
--

Now we can install libraries, frameworks, initialize projects based on templates, etc.
For now, we will use a package to test our project. I chose the library [Climate](https://github.com/thephpleague/climate), which is very cool and offers great options for colors, formatting, and other things in the terminal.

To use the library, we can require it from within our project folder as the documentation guides us:

```bash
$ composer require league/climate
```

When running the above command, we will get something like this:

```bash
rodrigo@localhost[~/repositories/php/hello-world-composer] $ composer require league/climate
Using version ^3.5 for league/climate
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 3 installs, 0 updates, 0 removals
  - Installing seld/cli-prompt (1.0.3): Downloading (100%)         
  - Installing psr/log (1.1.0): Loading from cache
  - Installing league/climate (3.5.0): Downloading (100%)         
Writing lock file
Generating autoload files
```

Composer installs all libraries and dependencies in the vendor/ folder. If you enter it, you will see numerous files there.

Now we can use the library. I will create a file called `main.php` and insert the following code:

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

use League\CLImate\CLImate;
$climate = new CLImate;
$climate->green('Composer!!!');
$climate->blue('Hello world');
```

Running it normally: **$ php main.php**, we will get the following output:

![climate output](https://i.ibb.co/pjZtSmh/print-climate.png)

#### Code Explanation

The code we wrote is very simple, and what makes the magic happen is our initial require:

```php
require_once __DIR__ . '/vendor/autoload.php';
```

When we include the `autoload.php` file, we are ready to use the installed libraries. Notice that we don't need to include all the dependencies the library needs because Composer takes care of it with an optimized autoload, doing all the magic behind the scenes. In the next article, we will use Composer to autoload classes that are outside Composer's context. We will create several classes that will use libraries in other directories, structuring the application.

Conclusion
--

This article was a simple and quick introduction to Composer. If you have never used it, this guide is enough to get you started.

Thanks and best regards!

#### References:

- [Composer](https://getcomposer.org/)
