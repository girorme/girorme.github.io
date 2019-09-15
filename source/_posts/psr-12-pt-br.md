---
title: Tradução PSR-12 PT-BR
date: 2019-09-04 23:37:02
tags:
---

> Até o presente momento não temos tradução da [PSR12](https://www.php-fig.org/psr/psr-12/). Então resolvi adiantar e submeter para alguns repos BR. :)

PSR-12: Estilo de codificação estendido
--

> As palavras chave "DEVE", "NÃO DEVE", "OBRIGATÓRIO", "TEM QUE", "NÃO TEM QUE", "DEVERIA", "NÃO DEVERIA", "RECOMENDADO", "PODE" e "OPCIONAL" existentes neste documento devem ser interpretadas como são descritas no RFC 2119.

## Visão geral
Essa especificação estende, expande e substitui a [PSR2](https://www.php-fig.org/psr/psr-2/). O estilo de codificação requer a adesão ao [PSR1](https://www.php-fig.org/psr/psr-1/), o padrão básico de codificação

Como a [PSR2](https://www.php-fig.org/psr/psr-2/), o intuito dessa especificação é reduzir a fricção cognitiva quando analisado/revisado código de autores diferentes. Isso é feito enumerando um conjunto de regras compartilhadas e expectativas de como formatar o código PHP.
Essa PSR procura fornecer uma maneira definida de implementação das ferramentas de estilo de codificação, os projetos podem declarar adesão e os desenvolvedores podem se relacionar facilmente entre diferentes projetos. Quando vários autores colaboram em vários projetos, a especificação ajuda a ter um conjunto de diretrizes a serem usadas entre todos esses projetos. Portanto, o benefício deste guia não está nas próprias regras, mas no compartilhamento dessas regras.

O [PSR2](https://www.php-fig.org/psr/psr-2/) foi aceito em 2012 e, desde então, várias alterações foram feitas no PHP, o que tem implicações nas diretrizes de estilo de codificação. Embora o [PSR2](https://www.php-fig.org/psr/psr-2/) seja muito abrangente da funcionalidade PHP existente no momento da redação, a nova funcionalidade é muito aberta à interpretação. Este PSR, portanto, procura esclarecer o conteúdo do [PSR2](https://www.php-fig.org/psr/psr-2/) em um contexto mais moderno, com novas funcionalidades disponíveis, e tornar as erratas à ligação ao [PSR2](https://www.php-fig.org/psr/psr-2/).

## Versões antigas da linguagem
Ao longo deste documento, todas as instruções PODEM ser ignoradas se não existirem nas versões do PHP suportadas pelo seu projeto.

## Exemplo
Este exemplo abrange algumas das regras abaixo como uma rápida visão geral:

```php
<?php

declare(strict_types=1);

namespace Vendor\Package;

use Vendor\Package\{ClassA as A, ClassB, ClassC as C};
use Vendor\Package\SomeNamespace\ClassD as D;

use function Vendor\Package\{functionA, functionB, functionC};

use const Vendor\Package\{ConstantA, ConstantB, ConstantC};

class Foo extends Bar implements FooInterface
{
    public function sampleFunction(int $a, int $b = null): array
    {
        if ($a === $b) {
            bar();
        } elseif ($a > $b) {
            $foo->bar($arg1);
        } else {
            BazClass::bar($arg2, $arg3);
        }
    }

    final public static function bar()
    {
        // corpo do método
    }
}
```

## 2 - Geral

### 2.1 Padrão Básico de codificação
- Códigos devem seguir PSR-1

- O termo 'StudlyCaps' na PSR-1 DEVE ser interpretado como PascalCase, onde a primeira letra de cada palavra é maiúscula.

### 2.2 Arquivos
- Todos arquivos PHP DEVEM usar somente o fim de linha LF (linefeed).

- Todos arquivos PHP DEVEM terminar com uma linha não branca, terminada com um único LF.

- A tag `?>` de fechamento DEVE ser omitida de arquivos contendo somente PHP.

### 2.3 Linhas
- NÃO DEVE existir um limite absoluto no comprimento da linha; O limite relativo DEVE ser de 120 caracteres;.

- As linhas`DEVERIAM ter 80 caracteres ou menos.

- NÃO DEVE existir um espaço em branco ao final das linhas.

- Linhas em branco PODEM ser adicionadas para melhorar a legibilidade e indicar blocos de códigos exceto onde explicitamente proibido.

- NÃO DEVE existir mais que uma declaração por linha.

### 2.4 Indentação
- Códigos DEVEM utilizar 4 espaços para indentação, não tabs.

### 2.5 Palavras-chave e tipos
- Todas palavras-chave e tipos do php [1][2] DEVEM ser minúsculas.

- Qualquer novo tipo e palavras-chave adicionado à versões futuras do PHP DEVEM ser em minúsculo

- Formas curtas de palavras-chave DEVEM ser usadas. Ex: `bool` ao invés de `boolean`, `int` ao invés de `integer`, etc.

## 3. `declare`, `namespace` e `use`

Cada bloco DEVE estar na ordem listada abaixo, embora os blocos que não são relevantes possam ser omitidos.

O cabeçalho de um arquivo PHP pode consistir de um número diferente de blocos. Se presente, cada um dos blocos abaixo DEVE ser separado por uma única linha em branco e NÃO DEVE conter uma linha em branco. Cada bloco DEVE estar na ordem listada abaixo, embora os blocos que não são relevantes podem ser omitidos.

- Tag `<?php` de abertura.
- Docblock em nível de arquivo.
- Uma ou mais declarações de `declare`.
- A declaração de `namespace` do arquivo.
- Uma ou mais instruções de `use` baseadas em classe.
- Uma ou mais instruções de `use` baseadas em funções.
- Uma ou mais instruções de `use` com base em constantes.
- O restante do código no arquivo.

Quando um arquivo contém uma mistura de HTML e PHP, qualquer uma das seções acima ainda pode ser usada. Nesse caso, eles DEVEM estar presentes no topo do arquivo, mesmo que o restante do código consista em uma tag PHP de fechamento e, em seguida, uma mistura de HTML e PHP.

As instruções de importação nunca DEVEM começar com uma barra invertida principal, pois sempre devem ser totalmente qualificadas.

O exemplo a seguir ilustra uma lista completa de todos os blocos:

```php
<?php

/**
 * Este arquivo contém um exemplo de estilo de código
 */

declare(strict_types=1);

namespace Vendor\Package;

use Vendor\Package\{ClassA as A, ClassB, ClassC as C};
use Vendor\Package\SomeNamespace\ClassD as D;
use Vendor\Package\AnotherNamespace\ClassE as E;

use function Vendor\Package\{functionA, functionB, functionC};
use function Another\Vendor\functionD;

use const Vendor\Package\{CONSTANT_A, CONSTANT_B, CONSTANT_C};
use const Another\Vendor\CONSTANT_D;

/**
 * FooBar é uma classe exemplo.
 */
class FooBar
{
    // ... código PHP ...
}
```

Os namespaces compostos com profundidade superior a dois NÃO DEVEM ser utilizados. Portanto, o seguinte é a profundidade máxima de composição permitida:

```php
<?php

use Vendor\Package\SomeNamespace\{
    SubnamespaceOne\ClassA,
    SubnamespaceOne\ClassB,
    SubnamespaceTwo\ClassY,
    ClassZ,
};
```

E o seguinte não seria permitido:

```php
<?php

use Vendor\Package\SomeNamespace\{
    SubnamespaceOne\AnotherNamespace\ClassA,
    SubnamespaceOne\ClassB,
    ClassZ,
};
```

Ao desejar declarar tipos estritos em arquivos contendo marcação fora das tags de abertura e fechamento do PHP, a declaração DEVE estar na primeira linha do arquivo e incluir uma tag PHP de abertura, a declaração de tipos estritos e a tag de fechamento.

Por exemplo:

```php
<?php declare(strict_types=1) ?>
<html>
<body>
    <?php
        // ... código PHP ...
    ?>
</body>
</html>
```
Declarações NÃO DEVEM conter espaços e DEVEM ser exatamente `declare(strict_types=1)` (com ponto e vírgula opcional).

As declarações de declare são permitidas e DEVEM ser formatadas como abaixo. Observe a posição das chaves e espaçamento:

```php
declare(ticks=1) {
    // código
}
```

## 4. Classes, Propriedades e Métodos
O termo "classe" refere-se a todas as classes, interfaces e traits.

Qualquer chave de fechamento NÃO DEVE ser seguida por nenhum comentário ou declaração na mesma linha.

Ao instanciar uma nova classe, os parênteses DEVEM estar sempre presentes, mesmo quando não houver argumentos passados para o construtor.

```php
new Foo();
```

### 4.1 `extends` e `implements`
As palavras chave extends e implements DEVEM ser declaradas na mesma linha que o nome da classe.

A chave de abertura para a classe DEVE seguir sua própria linha; a chave de fechamento para a classe DEVE seguir na próxima linha após o corpo.

As chaves de abertura DEVEM estar em sua própria linha e NÃO DEVEM ser precedidas ou seguidas por uma linha em branco.

As chaves de fechamento DEVEM estar em sua própria linha e NÃO DEVEM ser precedidas por uma linha em branco.

```php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements \ArrayAccess, \Countable
{
    // constants, properties, methods
}
```

Listas de `implements` e, no caso de interfaces, podem se estender por várias linhas, onde cada linha subseqüente é indentada uma vez. Ao fazer isso, o primeiro item da lista DEVE estar na próxima linha e DEVE haver apenas uma interface por linha.

```php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
    // contantes, propriedades, métodos
}
```

### 4.2 Utilizando traits

A palavra-chave `use` usada dentro das classes para implementar traits DEVE ser declarada na próxima linha após a chave de abertura.

```php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;
}
```

Cada trait individual importado para uma classe DEVE ser incluído um por linha e cada inclusão DEVE ter sua própria declaração de importação de uso.

```php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;
use Vendor\Package\SecondTrait;
use Vendor\Package\ThirdTrait;

class ClassName
{
    use FirstTrait;
    use SecondTrait;
    use ThirdTrait;
}
```

Quando a classe não tem nada após a declaração de `use`, a chave de fechamento da classe DEVE estar na próxima linha após a declaração de `use`.

```php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;
}
```

Caso contrário, DEVE ter uma linha em branco após a declaração de use

```php
<?php

namespace Vendor\Package;

use Vendor\Package\FirstTrait;

class ClassName
{
    use FirstTrait;

    private $property;
}
```

Ao usar os operadores `insteadof` e `as`, eles devem ser usados da seguinte maneira, seguindo indentação, espaçamento e novas linhas.

```php
<?php

class Talker
{
    use A, B, C {
        B::smallTalk insteadof A;
        A::bigTalk insteadof C;
        C::mediumTalk as FooBar;
    }
}
```

### 4.3 Propriedades e Constantes

A visibilidade DEVE ser declarada em todas as propriedades.

A visibilidade DEVE ser declarada em todas as constantes se o seu projeto versão mínima do PHP suportar visibilidades de constantes (PHP 7.1 ou posterior).

A palavra-chave `var` NÃO DEVE ser usada para declarar uma propriedade.

NÃO DEVE haver mais de uma propriedade declarada por declaração.

Os nomes de propriedade NÃO DEVEM ser prefixados com um único `_` para indicar visibilidade protegida ou privada. Ou seja, um prefixo com `_` explicitamente não tem significado.

DEVE haver um espaço entre a declaração de tipo e o nome da propriedade.

Uma declaração de propriedade se parece com o seguinte:

```php
<?php

namespace Vendor\Package;

class ClassName
{
    public $foo = null;
    public static int $bar = 0;
}
```

### 4.4 Metódos e Funções

A visibilidade DEVE ser declarada em todos os métodos.

Os nomes dos métodos NÃO DEVEM ser prefixados com um 'underline' para indicar visibilidade `protected` ou `private`. Ou seja, um prefixo com `_` não tem significado.

Os nomes de métodos e funções NÃO DEVEM ser declarados com espaço após o nome do método. A chave de abertura DEVE seguir sua própria linha, e a chave de fechamento DEVE seguir a próxima linha após o corpo. NÃO DEVE haver espaço após o parêntese de abertura e NÃO DEVE haver espaço antes do parêntese de fechamento.

Uma declaração de método se parece com o seguinte. Observe o posicionamento de parênteses, vírgulas, espaços e chaves:

```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function fooBarBaz($arg1, &$arg2, $arg3 = [])
    {
        // corpo do método
    }
}
```

A declaração de uma função se parece com o seguinte. Observe o posicionamento de parênteses, vírgulas, espaços e chaves:

```
<?php

function fooBarBaz($arg1, &$arg2, $arg3 = [])
{
    // function body
}
```

### 4.5 Argumentos de Métodos e Funções
Na lista de argumentos, NÃO DEVE haver um espaço antes de cada vírgula e DEVE haver um espaço após cada vírgula.

Argumentos de funções e métodos com valores padrão devem ir no final da lista de argumentos.

```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function foo(int $arg1, &$arg2, $arg3 = [])
    {
        // corpo do método
    }
}
```
Listas de argumentos PODEM ser divididas em várias linhas, onde cada linha subseqüente é indentada uma vez. Ao fazer isso, o primeiro item da lista DEVE estar na próxima linha e DEVE haver apenas um argumento por linha.

Quando a lista de argumentos é dividida em várias linhas, o parêntese de fechamento e a chave de abertura DEVEM ser colocados juntos em sua própria linha, com um espaço entre eles.

```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function aVeryLongMethodName(
        ClassTypeHint $arg1,
        &$arg2,
        array $arg3 = []
    ) {
        // corpo do método
    }
}
```

Quando você tem uma declaração `return` presente, DEVE haver um espaço após os dois pontos seguido pela declaração de tipo. Os dois pontos e a declaração DEVEM estar na mesma linha que a lista de argumentos entre parênteses, sem espaços entre os dois caracteres.

```php
<?php

declare(strict_types=1);

namespace Vendor\Package;

class ReturnTypeVariations
{
    public function functionName(int $arg1, $arg2): string
    {
        return 'foo';
    }

    public function anotherFunction(
        string $foo,
        string $bar,
        int $baz
    ): string {
        return 'foo';
    }
}
```

Nas declarações `nullable`, NÃO DEVE haver um espaço entre o ponto de interrogação e o tipo.

```php
<?php

declare(strict_types=1);

namespace Vendor\Package;

class ReturnTypeVariations
{
    public function functionName(?string $arg1, ?int &$arg2): ?string
    {
        return 'foo';
    }
}
```

Ao usar o operador de referência `&` antes de um argumento, NÃO DEVE haver um espaço depois dele, como no exemplo anterior.

NÃO DEVE haver um espaço entre o operador three dot `...` e o nome do argumento:

```php
public function process(string $algorithm, ...$parts)
{
    // processando
}
```

Ao combinar o operador de referência `&` e o operador three dot `...`, NÃO DEVE haver nenhum espaço entre os dois:

```php
public function process(string $algorithm, &...$parts)
{
    // processando
}
```

### 4.6 `abstract`, `final` e `static`
Quando presentes, as declarações `abstract` e `final` DEVEM preceder a declaração de visibilidade.

Quando presente, a declaração `static` DEVE vir após a declaração de visibilidade.

```php
<?php

namespace Vendor\Package;

abstract class ClassName
{
    protected static $foo;

    abstract protected function zim();

    final public static function bar()
    {
        // corpo do método
    }
}
```

### 4.7 Chamada de Métodos e Funções
Ao fazer uma chamada de método ou função, NÃO DEVE haver um espaço entre o nome do método ou da função e o parêntese de abertura, NÃO DEVE haver um espaço após o parêntese de abertura e NÃO DEVE haver espaço antes do parêntese de fechamento. Na lista de argumentos, NÃO DEVE haver um espaço antes de cada vírgula e DEVE haver um espaço após cada vírgula.

```php
<?php

bar();
$foo->bar($arg1);
Foo::bar($arg2, $arg3);
```

Listas de argumentos podem ser divididas em várias linhas, onde cada linha subseqüente é indentada uma vez. Ao fazer isso, o primeiro item da lista DEVE estar na próxima linha e DEVE haver apenas um argumento por linha. Um único argumento dividido em várias linhas (como pode ser o caso de uma função anônima ou array) não constitui a divisão da própria lista de argumentos.

```
<?php

$foo->bar(
    $longArgument,
    $longerArgument,
    $muchLongerArgument
);
```

```php
<?php

somefunction($foo, $bar, [
  // ...
], $baz);

$app->get('/hello/{name}', function ($name) use ($app) {
    return 'Hello ' . $app->escape($name);
});
```

## 5. Estruturas de controle
As regras gerais de estilo para estruturas de controle são as seguintes:

- DEVE haver um espaço após a palavra-chave da estrutura de controle
- NÃO DEVE haver um espaço após o parêntese de abertura
- NÃO DEVE haver espaço antes do fechamento de parênteses
- DEVE existir um espaço entre o parêntese de fechamento e a chave de abertura
- O corpo da estrutura DEVE ser indentado uma vez
- O corpo DEVE estar na próxima linha após a chave de abertura
- A chave de fechamento DEVE estar na próxima linha após o corpo

O corpo de cada estrutura DEVE ser delimitado por chaves. Isso padroniza a aparência das estruturas e reduz a probabilidade de introdução de erros à medida que novas linhas são adicionadas ao corpo.

### 5.1 if, elseif, else
Uma estrutura `if` se parece com o seguinte. Observe o posicionamento de parênteses, espaços e chaves; e que `else` e `elseif` estão na mesma linha que a chave de fechamento do corpo anterior.

```php
<?php

if ($expr1) {
    // if
} elseif ($expr2) {
    // elseif
} else {
    // else;
}
```
A palavra-chave `elseif` DEVERIA ser utilizada ao invés de `else if`, para que todas as palavras-chave de controle se pareçam com uma só palavra.

Expressões entre parênteses PODEM ser divididas em várias linhas, onde cada linha subseqüente é indentada pelo menos uma vez. Ao fazer isso, a primeira condição DEVE estar na próxima linha. O parêntese de fechamento e a chave de abertura DEVEM ser colocados juntos em sua própria linha, com um espaço entre eles. Operadores booleanos entre condições DEVEM estar sempre no início ou no final da linha, não uma mistura de ambos.

```php
<?php

if (
    $expr1
    && $expr2
) {
    // corpo do if
} elseif (
    $expr3
    && $expr4
) {
    // corpo do elseif
}
```

### 5.2 `switch`, `case`
Uma estrutura de `switch` é semelhante à seguinte. Observe o posicionamento de parênteses, espaços e chaves. A declaração de `case` DEVE ser indentada uma vez no `switch`, e a palavra-chave `break` (ou outras palavras-chave de término) DEVEM ser identadas no mesmo nível que o corpo do `case`. DEVE haver um comentário como `// no break` quando não há um `break` em um corpo de `case` não vazio

```php
<?php

switch ($expr) {
    case 0:
        echo 'First case, with a break';
        break;
    case 1:
        echo 'Second case, which falls through';
        // no break
    case 2:
    case 3:
    case 4:
        echo 'Third case, return instead of break';
        return;
    default:
        echo 'Default case';
        break;
}
```

Expressões entre parênteses PODEM ser divididas em várias linhas, onde cada linha subseqüente é indentada pelo menos uma vez. Ao fazer isso, a primeira condição DEVE estar na próxima linha. O parêntese de fechamento e a chave de abertura DEVEM ser colocados juntos em sua própria linha, com um espaço entre eles. Operadores booleanos entre condições DEVEM estar sempre no início ou no final da linha, não uma mistura de ambos.

```php
<?php

switch (
    $expr1
    && $expr2
) {
    // corpo do switch
}
```

### 5.3 while, do while
Uma declaração `while` se parece com o seguinte. Observe o posicionamento de parênteses, espaços e chaves.

```php
<?php

while ($expr) {
    // corpo do while
}
```

Expressões entre parênteses PODEM ser divididas em várias linhas, onde cada linha subseqüente é indentada pelo menos uma vez. Ao fazer isso, a primeira condição DEVE estar na próxima linha. O parêntese de fechamento e a chave de abertura DEVEM ser colocados juntos em sua própria linha, com um espaço entre eles. Operadores booleanos entre condições DEVEM estar sempre no início ou no final da linha, não uma mistura de ambos.

```php
<?php

while (
    $expr1
    && $expr2
) {
    // structure body
}
```

Da mesma forma, uma declaração `do while` se parece com o seguinte. Observe o posicionamento de parênteses, espaços e chaves.

```php
<?php

do {
    // corpo da estrutura;
} while ($expr);
```

Expressões entre parênteses PODEM ser divididas em várias linhas, onde cada linha subseqüente é indentada pelo menos uma vez. Ao fazer isso, a primeira condição DEVE estar na próxima linha. Operadores booleanos entre condições DEVEM estar sempre no início ou no final da linha, não uma mistura de ambos.

```php
<?php

do {
    // corpo da estrutura;
} while (
    $expr1
    && $expr2
);
```

### 5.4 for
Uma declaração `for` se parece com o seguinte. Observe o posicionamento de parênteses, espaços e chaves.

```php
<?php

for ($i = 0; $i < 10; $i++) {
    // corpo do for
}
```

Expressões entre parênteses podem ser divididas em várias linhas, onde cada linha subseqüente é indentada pelo menos uma vez. Ao fazer isso, a primeira expressão DEVE estar na próxima linha. O parêntese de fechamento e a chave de abertura DEVEM ser colocados juntos em sua própria linha, com um espaço entre eles.

```php
<?php

for (
    $i = 0;
    $i < 10;
    $i++
) {
    // corpo do for
}
```

### 5.5 `foreach`
Uma declaração `foreach` é semelhante à seguinte. Observe o posicionamento de parênteses, espaços e chaves.

```php
<?php

foreach ($iterable as $key => $value) {
    // corpo foreach
}
```

### 5.6 `try`, `catch` e `finally`
Um bloco `try-catch-finally` se parece com o seguinte. Observe o posicionamento de parênteses, espaços e chaves.

```php
<?php

try {
    // corpo try
} catch (FirstThrowableType $e) {
    // corpo catch
} catch (OtherThrowableType | AnotherThrowableType $e) {
    // corpo catch
} finally {
    // corpo finally
}
```

### 6. Operadores
As regras de estilo para os operadores são agrupadas por aridade (o número de operandos que eles usam).

Quando o espaço é permitido em torno de um operador, vários espaços podem ser usados para fins de legibilidade.

Todos os operadores não descritos aqui são deixados indefinidos.

### 6.1. Operadores unários
Os operadores de incremento / decremento NÃO DEVEM ter nenhum espaço entre o operador e o operando.

Os operadores de conversão de tipo NÃO DEVEM ter nenhum espaço entre parênteses:

```php
$intValue = (int) $input;
```

### 6.2. Operadores binários
Todos os operadores aritméticos, de comparação, de atribuição, bit a bit, lógicos, de string e de tipo binário DEVEM ser precedidos e seguidos por pelo menos um espaço:

```php
if ($a === $b) {
    $foo = $bar ?? $a ?? $b;
} elseif ($a > $b) {
    $foo = $a + $b * $c;
}
```

### 6.3. Operadores ternários
O operador condicional, também conhecido simplesmente como operador ternário, DEVE ser precedido e seguido por pelo menos um espaço em torno dos caracteres: `?` e `:`

```php
$variable = $foo ? 'foo' : 'bar';
```

Quando o operando do meio do operador condicional é omitido, o operador DEVE seguir as mesmas regras de estilo que outros operadores de comparação binária:

```php
$variable = $foo ?: 'bar';
```

## 7. Closures
Closures DEVEM ser declarados com um espaço após a palavra-chave `function` e um espaço antes e depois da palavra-chave `use`.

A chave de abertura DEVE seguir na mesma linha e a chave de fechamento DEVE seguir na próxima linha após o corpo.

NÃO DEVE haver um espaço após o parêntese de abertura da lista de argumentos ou da lista de variáveis, e NÃO DEVE haver um espaço antes do parêntese de fechamento da lista de argumentos ou da lista de variáveis.

Na lista de argumentos e na lista de variáveis, NÃO DEVE haver um espaço antes de cada vírgula e DEVE haver um espaço após cada vírgula.

Argumentos de fechamento com valores padrão DEVEM ir no final da lista de argumentos.

Se um tipo de retorno estiver presente, DEVE seguir as mesmas regras que as funções e métodos normais; se a palavra-chave use estiver presente, os dois pontos DEVE seguir a lista de uso entre parênteses sem espaços entre os dois caracteres.

Uma declaração de fechamento é semelhante à seguinte. Observe o posicionamento de parênteses, vírgulas, espaços e chaves:

```php
<?php

$closureWithArgs = function ($arg1, $arg2) {
    // corpo
};

$closureWithArgsAndVars = function ($arg1, $arg2) use ($var1, $var2) {
    // corpo
};

$closureWithArgsVarsAndReturn = function ($arg1, $arg2) use ($var1, $var2): bool {
    // corpo
};
```
Listas de argumentos e listas de variáveis PODEM ser divididas em várias linhas, onde cada linha subseqüente é indentada uma vez. Ao fazer isso, o primeiro item da lista DEVE estar na próxima linha e DEVE haver apenas um argumento ou variável por linha.

Quando a lista final (seja de argumentos ou variáveis) é dividida em várias linhas, o parêntese de fechamento e a chave de abertura DEVEM ser colocados juntos em sua própria linha, com um espaço entre eles.

A seguir, exemplos de fechamentos com e sem listas de argumentos e listas de variáveis divididas em várias linhas.

```php
<?php

$longArgs_noVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) {
   // corpo
};

$noArgs_longVars = function () use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // corpo
};

$longArgs_longVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // corpo
};

$longArgs_shortVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use ($var1) {
   // corpo
};

$shortArgs_longVars = function ($arg) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // corpo
};
```
Observe que as regras de formatação também se aplicam quando o fechamento é usado diretamente em uma chamada de função ou método como argumento.

```php
<?php

$foo->bar(
    $arg1,
    function ($arg2) use ($var1) {
        // body
    },
    $arg3
);
```

### 8. Classes anônimas
Classes anônimas DEVEM seguir as mesmas diretrizes e princípios que os fechamentos na seção acima.

```php
<?php

$instance = new class {};
```
A chave de abertura PODE estar na mesma linha que a palavra-chave `class`, desde que a lista de `implements` de interfaces  não seja finalizada. Se a lista de interfaces terminar, a chave DEVE ser colocada na linha imediatamente após a última interface.

```php
<?php

// Chave de abertuna na mesma linha
$instance = new class extends \Foo implements \HandleableInterface {
    // Conteúdo da classe
};

// Chave de abertura na  próxima linha
$instance = new class extends \Foo implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
    // Conteúdo da classe
};
```

- [1] http://php.net/manual/en/reserved.keywords.php
- [2] http://php.net/manual/en/reserved.other-reserved-words.php
