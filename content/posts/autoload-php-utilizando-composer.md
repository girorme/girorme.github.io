+++
author = "Girorme"
title = "Autoload no php Utilizando Composer"
date = "2019-09-01"
description = "Aprenda como criar um projeto utilizando composer e o padrão PSR4 para autoloading"
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

Temos o php antes e depois do composer e você precisa saber como gerenciar seu projeto utilizando essa ferramenta incrível :)

## TL;DR
> Aprenda como criar um projeto utilizando composer e o padrão PSR4 para autoloading

### Era tudo mato
Você certamente já viu ou já utilizou as funções `require`/`require_once` ou até mesmo `include` enquanto programava com PHP.
Funções essas que carregam todo o conteúdo do arquivo solicitado em memória, fazendo com que em determinado momento fique inviável a utilização destas de forma descontrolada em projetos de média/larga escala.

Felizmente com o passar do tempo *[especificações foram surgindo](https://www.php-fig.org/)* e a forma que programamos em php (ou deveriamos programar... rs) mudou muito. 
Padrões foram adotados e ganhamos maturidade/[interoperabilidade](https://pt.wikipedia.org/wiki/Interoperabilidade) no desenvolvimento.

Dentre essas especificações temos algumas que dizem respeito ao autoloading de classes (PSR-0 e PSR-4). A PSR-0 hoje se encontra depreciada e como substituição temos a ¹[PSR-4](https://www.php-fig.org/psr/psr-4/) que nos da um padrão de como devemos estruturar nosso projeto para ter um autoload "mágico"/eficiente/perfomático e por demanda (Os arquivos só são carregados em memória quando precisamos deles!).
Para isso temos algumas regras bem definidas que iremos colocar em prática criando um projeto via composer ([Já escrevi brevemente sobre o composer se não souber do que se trata...](http://girorme.github.io/2017/07/23/gerenciando-bibliotecas-php-composer/)).

### Criando o projeto com composer
Uma vez que você já possui o composer pronto para uso podemos criar o projeto:

{{< highlight bash "linenos=table" >}}
$ mkdir autoload-com-composer && cd autoload-com-composer
$ composer init
{{< / highlight >}}

O comando `composer init` nos pede algumas informações sobre o projeto, como nome do pacote, versão, licença etc.
Deixei da seguinte forma:

{{< highlight text >}}
Package name (<vendor>/<name>) [rodrigo/autoload-com-composer]: girorme/autoload-com-composer
Description []:
Author [girorme <rodrigo.girorme@gmail.com>, n to skip]:
Minimum Stability []:
Package Type (e.g. library, project, metapackage, composer-plugin) []: project
License []: MIT
Would you like to define your dependencies (require) interactively [yes]? no
Would you like to define your dev dependencies (require-dev) interactively [yes]? no
{{< / highlight >}}

Feito isso o arquivo `composer.json` é criado e podemos iniciar nosso experimento.
Iremos definir agora a estrutura de pastas/arquivos que a especificação espera.
A estrutura fica o seguinte:

{{< highlight text >}}
- src/
  - App/
- composer.json
- index.php
{{< / highlight >}}

Dentro de `src/` criamos a pasta `App` que atuará como nosso namespace. Essa pasta será nosso "vendor namespace" ou seja, caso criarmos um arquivo `Battle.php` dentro de `src\App` chamaremos ele utilizando: `use App\Battle`, agora se queremos um subnamespace `Char` precisamos criar uma pasta dentro de app: `src/App/Char`. E para chamar uma classe dentro de Char que se chama `Ryu` com nome de arquivo `Ryu.php` por exemplo? Simples: `use App\Char\Ryu`.

Antes disso para que o composer faça o autoload das classes precisamos alterar o arquivo `composer.json` mapeando esse namespace para que ele faça o trabalho pesado. O arquivo ficará da seguinte forma:

{{< highlight bash "linenos=table" >}}
{
    "name": "girorme/autoload-com-composer",
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
{{< / highlight >}}

Antes de entrar no código é necessário saber que a especificação exige uma estrutura bem definida como já mencionei e vamos representar isso nos próximos exemplos.

Após alterar o arquivo `composer.json` é necessário rodar o comando `dump-autoload` para que nossa estrutura seja identificada:

{{< highlight text >}}
$ composer dump-audoload
{{< / highlight >}}

Agora podemos criar nossas classes e ver o autoloading funcionando na prática. Vamos criar algumas classes para representar personanges do street fighter e outras classes para representar alguns atributos. Vamos antes ver o autoload funcionando.
Criaremos inicialmente um arquivo chamado `Battle.php` dentro de `src/App`:

A classe `Battle.php`:

{{< highlight php "linenos=table" >}}
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

    public function startBatle()
    {
        echo sprintf('%s Fight!' . PHP_EOL, $this->fightTitle);
    }
}
{{< / highlight >}}

Lembre-se que definimos nosso "vendor namespace" como `App` então precisamos referenciá-lo no inicio do arquivo.

Agora vamos criar nosso ponto de inicio dentro do arquivo `index.php` fazendo com que o composer traga os arquivos/classes solicitados:

{{< highlight php "linenos=table" >}}
# index.php

<?php

require_once __DIR__ . '/vendor/autoload.php';

use App\Battle;

$battle = new Battle('Street Fighter!');
$battle->start();
{{< / highlight >}}

Após incluirmos apenas o arquivo `/vendor/autoload.php` que é gerado pelo composer teremos todas as classes solicitadas por demanda.

A saída do código acima fica da seguinte forma:

{{< highlight bash >}}
$ php index.php
Street Fighter! Fight!
{{< / highlight >}}

Pronto! Você já tem o autoload funcionando. Podemos agora colocar outros pontos em prática, exemplificando por exemplo como podemos ter sub-diretórios.

Vamos adicionar alguns personagens e atributos à batalha. Deixarei aqui a versão final do arquivo `index.php` mas caso queira conferir o código fonte do exemplo -> [composer-autoload-example](https://github.com/girorme/composer-autoload-example).

A nossa estrutura final ficará da seguinte forma:

{{< highlight text >}}
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
{{< / highlight >}}

Nosso objetivo basicamente é:
> Criar dois lutadores 
  |> Adicionar atributos à eles 
  |> Criar uma batalha
  |> Adicionar os lutadores à batalha
  |> Mostrar os lutadores e atributos

Versão final do arquivo index.php:

{{< highlight php "linenos=table" >}}
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
{{< / highlight >}}

Saída:

{{< highlight bash "linenos=table" >}}
$ php index.php
Street Fighter! Fight!
Char: Ryu, Power: Hadoooouken
Char: Ken, Power: Shooooooryuken
{{< / highlight >}}

Veja como é simples a utilização das classes:

{{< highlight php "linenos=table" >}}
use App\Battle;
use App\Char\{Ryu, Ken};
use App\Attributes\{Hadouken, Shoryuken};
{{< / highlight >}}

> ³ * **OBS**: a notação `App\Char\{Ryu, Ken};` é uma característa do PHP7 que facilita a utilização de múltiplas classes, eu poderia representar o mesmo utilizando:

{{< highlight php "linenos=table" >}}
use App\Char\Ryu;
use App\Char\Ken;
{{< / highlight >}}

Agora confira no código fonte que temos tanto uma classe no primeiro nível que é o caso de `App\Battle` como subnamespaces que é o caso dos chars e dos atributos `App\Char\Ryu` e `App\Attributes\Hadouken`. Abra esses arquivos veja como foi feito, mude a estrutura e entenda como funciona a psr-4! :)

Turma, é isso!


## Referências

- [1] [PSR-4](https://www.php-fig.org/psr/psr-4/)
- [2] [Composer](https://getcomposer.org/)
- [3] [PHP Group use declarations](https://www.php.net/manual/pt_BR/language.namespaces.importing.php#language.namespaces.importing.group)

