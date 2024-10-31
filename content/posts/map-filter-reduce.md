+++
author = "girorme"
title = "Código Limpo e Elegante com Map, Filter e Reduce"
date = "2024-10-31"
description = "Descubra como map, filter e reduce podem transformar seu código: elegância, expressividade e concisão em cada linha. Simplifique suas operações e escreva menos, fazendo mais!"
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

Já ouviu falar de map, filter e reduce? Se já conhece, certamente está
escrevendo código mais expressivo. Se não conhece, aqui vamos descobrir porque usar e e de onde vem essa flexibilidade...

![fn](/images/functional_programming_concept.png)

A maioria de nós começa programando no estilo imperativo: dando instruções passo a passo, dizendo ao código o que ele deve fazer. Você cria loops, controla variáveis e escreve aquele monte de detalhes para chegar no resultado final. Mas sabia que existe um jeito mais elegante de fazer isso? Com map, filter e reduce, dá pra simplificar o processo. Neste post, vamos comparar o jeito tradicional com essas funções, mostrando como elas podem deixar seu código mais limpo e enxuto (exemplos em python).


### 1. Exemplo com `map`

Imagine que você tem uma lista de números e quer dobrar o valor de cada um.

#### Imperativo
No estilo imperativo, você teria que criar uma nova lista, iterar sobre os números e adicionar o dobro de cada um na nova lista. Algo assim:

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
doubled = []
for num in nums:
    doubled.append(num * 2)
{{< / highlight >}}

Aqui você está dizendo ao programa **como** fazer: "Crie uma lista vazia, faça um loop, dobre os números e insira um por um."

#### Funcional com `map`
Agora, usando `map`, você expressa **o que** você quer: "Dobre cada número dessa lista."

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
doubled = map(lambda x: x * 2, nums)
{{< / highlight >}}


Repare que você não precisa falar nada sobre a lista vazia ou sobre como percorrer os itens. O `map` cuida disso. Isso torna o código **mais curto** e **focado** na transformação, não no processo. *Menos boilerplate, mais resultado!*

### 2. Exemplo com `filter`

Agora, imagine que você quer pegar apenas os números pares dessa lista.

#### Imperativo
Aqui, você precisaria fazer mais um loop, verificando cada número, e, se for par, adicioná-lo a uma nova lista:

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
evens = []
for num in nums:
    if num % 2 == 0:
        evens.append(num)
{{< / highlight >}}


Estamos novamente dizendo ao programa **como** deve proceder: "Verifique cada número, veja se é par e coloque na lista."

#### Funcional com `filter`
Com `filter`, é bem direto: "Selecione os pares."

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
evens = filter(lambda x: x % 2 == 0, nums)
{{< / highlight >}}


A mágica aqui é que você foca apenas na **condição** de filtragem, não no controle de fluxo. Além de deixar o código mais **limpo**, isso evita erros comuns, como esquecer de adicionar o item na lista ou escrever lógica confusa no loop.

### 3. Exemplo com `reduce`

Vamos dizer que você quer somar todos os números da lista. No estilo imperativo, você teria que declarar uma variável acumuladora e iterar sobre os números, somando um por um.

#### Imperativo
Aqui vai a abordagem tradicional:

{{< highlight python "linenos=table" >}}
nums = [1, 2, 3, 4]
total = 0
for num in nums:
    total += num
{{< / highlight >}}


Você está, de novo, **dizendo como** fazer. E aqui já aparece a necessidade de uma variável extra, `total`, para manter o controle.

#### Funcional com `reduce`
Com `reduce`, você expressa diretamente: "Reduza essa lista para um único valor somando os itens."

{{< highlight python "linenos=table" >}}
from functools import reduce
nums = [1, 2, 3, 4]
total = reduce(lambda acc, x: acc + x, nums)
{{< / highlight >}}


Olha que lindo: tudo o que você precisa está numa única linha. O `reduce` pega a lista e vai acumulando o valor ao longo da sequência, e você não precisa se preocupar com variáveis adicionais nem com o controle do loop.

### Por que usar `map`, `filter` e `reduce` é elegante?

O grande diferencial das funções funcionais em comparação com o estilo imperativo:

1. **Expressividade**: Usar essas funções permite que você descreva o **intento** do código, ao invés de detalhar **passo a passo como** a operação deve ser feita. O código fica mais próximo da forma como pensamos sobre o problema, o que melhora a **legibilidade**.

2. **Abstração do controle de fluxo**: Você não precisa se preocupar com loops, índices, variáveis temporárias e tudo aquilo que enche o código de detalhes desnecessários. Isso minimiza a chance de erros e melhora a **manutenção**.

3. **Composição fácil**: O estilo funcional permite **compor operações** de maneira intuitiva. Você pode encadear `map`, `filter` e `reduce` sem perder clareza:

   {{< highlight python "linenos=table" >}}
   nums = [1, 2, 3, 4]
   total = reduce(lambda acc, x: acc + x, filter(lambda x: x % 2 == 0, map(lambda x: x * 2, nums)))
   {{< / highlight >}}

   
   Aqui, você está dobrando os números, filtrando os pares e somando tudo numa única expressão. Em estilo imperativo, isso exigiria vários loops e variáveis, o que complicaria bem mais o código.

4. **Imutabilidade e menos estado**: Essas funções não alteram os dados originais (a menos que você force). Isso te protege de problemas difíceis de depurar, como mudanças inesperadas de estado, algo comum no estilo imperativo. No mundo funcional, os dados são sagrados, e as funções tratam deles sem efeitos colaterais.

### Outras linguagens

É importante ressaltar que você não precisa usar uma linguagem funcional como Elixir para aproveitar funções como `map`, `filter` e `reduce` (utilizamos python nos exemplos). Essas funções estão disponíveis em diversas linguagens de programação. Vamos ver como implementar os mesmos exemplos em PHP e JavaScript.

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

Esses exemplos mostram que, independentemente da linguagem, o conceito de funções de alta ordem (funções que recebem ou retornam outras funções) é amplamente aplicável, permitindo que você escreva código mais conciso e expressivo.

### Resumo: de imperativo a elegante

Quando você vê código imperativo, está focado em **como** a operação deve ser feita, o que leva a mais código e mais detalhes de implementação. Com `map`, `filter` e `reduce`, você diz **o que** quer fazer, e deixa que a linguagem se encarregue dos detalhes. Isso resulta em código mais enxuto, mais legível e menos propenso a erros.

Essas funções são como a trilha sonora perfeita para manipulação de dados: discretas, eficientes e fazem o trabalho com classe!