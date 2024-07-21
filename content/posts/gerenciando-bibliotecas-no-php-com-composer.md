+++
author = "girorme"
title = "Gerenciando bibliotecas no PHP com composer"
date = "2017-07-23"
description = "O composer é indispensável no desenvolvimento de aplicações maduras no php"
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

O composer é um gerenciador de dependências para o PHP, também podemos utiliza-lo também para gerenciamento de autoloading e outras coisas

## Instalação composer

#### Linux
[Composer linux](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos)

#### Windows
[Composer windows](https://getcomposer.org/doc/00-intro.md#installation-windows)


Se tudo deu certo, você pode executar o comando:

```text
$ composer
```

E várias opções serão exibidas

Onde estão as bibliotecas do composer?
--

Da forma mais simples, quando gerenciamos bibilotecas pelo composer, estamos buscando essas bibliotecas no [Packagist](https://packagist.org/).



Criando nosso hello-world com composer
--

Criação da pasta

```bash
$ mkdir hello-world-composer
```

 Agora iremos inicializar o composer dentro da nossa pasta

```bash
$ composer init
```

Ao executar o comando acima seremos questionados sobre, nome do autor, tipo do projeto, licença, etc.

Preencha todas as informações conforme orientado e quando chegar no momento onde o composer pergunta se quer definir as bibliotecas do projeto, digite N para podermos fazer isso manualmente. Também será perguntado se queremos fazer o require-dev que são as depêndencias utilizadas para desenvolvimento local/testes, etc. Iremos entender em breve :)

Ao final do processo de gerar o arquivo `composer.json` ele estará parecido com o abaixo:

```json
{
    "name": "girorme/hello-wolrd-composer",
    "description": "Projeto teste composer",
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

A partir desse momento já podemos fazer uma baguncinha com o composer.

Instalando bibliotecas
--

Agora podemos instalar bibliotecas, frameworks, inicializar projetos baseados em templates, etc.
Por enquanto iremos utilizar um pacote para testar nosso projeto. Escolhi a bibiloteca [Climate](https://github.com/thephpleague/climate) que é muito legal e nos da opções muito legais para cores, formatação e outras coisas no terminal.

Para usar a biblioteca podemos dar o require de dentro da nossa pasta do projeto conforme a documentação nos orienta:

```bash
$ composer require league/climate
```

Ao executar o comando acima teremos algo parecido com:

```bash
rodrigo@localhost[~/repositorios/php/hello-world-composer] $ composer require league/climate
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

O composer instala todas as bibliotecas e dependências na pasta vendor/, se entrar nela verá os inúmeros arquivos lá existentes


Agora podemos utilizar a biblioteca. Irei criar um arquivo chamado `main.php` e inserir o código a seguir:

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

use League\CLImate\CLImate;
$climate = new CLImate;
$climate->green('Commposer!!!');
$climate->blue('Hello world');
```

Executando normalmente: **$ php main.php**, teremos a saída abaixo:

![saida climate](https://i.ibb.co/pjZtSmh/print-climate.png)


#### Explicação do código

O código que escrevemos foi bem simples e o que faz a mágica acontecer é o nosso require inicial:

```php
require_once __DIR__ . '/vendor/autoload.php';
```

Quando incluimos o arquivo `autoload.php` estamos prontos para utilizar as bibliotecas instaladas. Repare que não precisamos incluir todas as dependências que a biblioteca precisa porque o composer se encarrega disso com um autoload otimizado fazendo toda a mágica por trás. No próximo artigo iremos utilizar o composer para auto carregar classes que estão fora do contexto do composer ou seja, criaremos várias classes que utilizarão as bibliotecas em outros diretórios, estruturando a aplicação.



Conclusão
--

Esse artigo foi uma introdução simples e rápida sobre composer. Se você nunca utilizou o mesmo esse guia é o suficiente para começar a utilizar.

Obregado e forte abraço!


#### Referências:

- [Composer](https://getcomposer.org/)