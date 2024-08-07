+++
author = "girorme"
title = "GO para devs PHP: O tal do Json"
date = "2021-02-06"
description = "Aprenda como manipular json na linguagem go"
tags = [
    "php",
    "go"
]
categories = [
    "php",
    "go"
]
+++

Da série go para devs php:
Existe uma diferença notável em como as linguagens tratam json e
se você está estudando GO e não achou os galãs `json_encode` e `json_decode` esse artigo é pra você!

## Index
- [Intro](#)
- [Iniciando o projeto](#)
- [Decoding Json (Unmarshal)](#)
- [Encoding Json (Marshal)](#)
- [Struct tags](#)
- [Considerações](#)
- [Ref](#)
    
### Intro
O pacote `encoding/json` da biblioteca padrão do go possui todas as funcionalidades necessárias para encodar/decodar json. Quando lidamos com json dentro do go por se tratar de uma linguagem com tipagem forte precisamos "mapear" os campos de um json para uma estrutura que faça sentido

![jason](/images/jason.jpg)

## Iniciando o projeto
Vamos criar um projeto simples:

```text
$ mkdir jsonbyexample; cd $_
$ go mod init jsonbyexample
```

Com isso podemos criar um arquivo `main.go` com algumas coisas:

{{< highlight go "linenos=table" >}}
package main

var (
    jsonPerson  = `{"name": "Rodrigo", "age": 28}`
    jsonTech    = `{"project": "foo/bar", "stack": ["go", "elk", "elixir"]}`
    jsonCountry = `{
        "location": "Brazil", 
        "goods" :[
            {"day": "Wednesday", "food": "Feijoada e caipirinha"},
            {"day": "Monday", "food": "Virado à paulista"},
            {"day": "Friday", "food": "Arroz com molho de camarão e Merluza"}
        ]
    }`
)

{{< / highlight >}}

Criamos alguns formatos jsons para explorar:

- Chave e valor
- Chave e valor + array
- Chave e valor + array de objetos


## Decoding json (Unmarshal)

Vamos começar convertendo os jsons para trabalha-los no go. Para isso precisamos criar estruturas compatíveis. Vamos começar com o `jsonPerson`:

{{< highlight go "linenos=table" >}}
type Person struct {
    Name string
    Age  int
}
{{< / highlight >}}

Aqui mapeamos os campos `Name` como `string` e `Age` como `int` que é de fato o tipo contido no json

> Importante: repare que seguindo a convenção da linguagem a primeira letra de cada campo está em maíusculo indicando que esse campo está sendo exportado e que pode ser acessado posteriormente

Agora para trabalhar com o dado:

{{< highlight go "linenos=table" >}}
package main

import (
    "encoding/json"
    "fmt"
)

var (
    jsonPerson  = `{"name": "Rodrigo", "age": 28}`
    jsonTech    = `{"project": "foo/bar", "stack": ["go", "elk", "elixir"]}`
    jsonCountry = `{
        "location": "Brazil", 
        "goods" :[
            {"day": "Wednesday", "food": "Feijoada e caipirinha"},
            {"day": "Monday", "food": "Virado à paulista"},
            {"day": "Friday", "food": "Arroz com molho de camarão e Merluza"}
        ]
    }`
)

type Person struct {
    Name string
    Age  int
}

func main() {
    // ==== Person ====
    var person Person
    json.Unmarshal([]byte(jsonPerson), &person)
    fmt.Printf("[Person] Name: %s - Age: %d\n", person.Name, person.Age)
}
{{< / highlight >}}

Aqui fazemos a utilização da função `json.Unmarshal` que espera o dado a ser decodado no formato `[]byte` e a refêrencia à variavel a ser preenchida `&person`

E para acessar os dados usamos dotNotation como no exemplo acima

Resultado:
```text
girorme@machine ~/repositorios/go/jsonbyexample $ go run .
[Person] Name: Rodrigo - Age: 28
```

Vamos agora mapear o json `jsonTech` que uma das chaves é um array de strings:

{{< highlight go "linenos=table" >}}
type Tech struct {
    Project string
    Stack   []string
}
{{< / highlight >}}

Simplesmente indicamos que a chave Stack é um array de strings. O decoding continua simples:

{{< highlight go "linenos=table" >}}
package main

import (
    "encoding/json"
    "fmt"
)

var (
    jsonPerson  = `{"name": "Rodrigo", "age": 28}`
    jsonTech    = `{"project": "foo/bar", "stack": ["go", "elk", "elixir"]}`
    jsonCountry = `{
        "location": "Brazil", 
        "goods" :[
            {"day": "Wednesday", "food": "Feijoada e caipirinha"},
            {"day": "Monday", "food": "Virado à paulista"},
            {"day": "Friday", "food": "Arroz com molho de camarão e Merluza"}
        ]
    }`
)

type Person struct {
    Name string
    Age  int
}

type Tech struct {
    Project string
    Stack   []string
}

func main() {
    // ==== Person ====
    var person Person
    json.Unmarshal([]byte(jsonPerson), &person)
    fmt.Printf("[Person] Name: %s - Age: %d\n", person.Name, person.Age)

    // ==== Tech ====
    var tech Tech
    json.Unmarshal([]byte(jsonTech), &tech)
    fmt.Printf("[Tech] Project: %s - Stack: %s\n", tech.Project, tech.Stack)
}
{{< / highlight >}}

Resultado:

```text
girorme@machine ~/repositorios/go/jsonbyexample $ go run .
[Person] Name: Rodrigo - Age: 28
[Tech] Project: foo/bar - Stack: [go elk elixir]
```

Vamos agora mapear o json `jsonCountry` que umas das chaves é um array de objetos. Para alcançar esse mapping precisamos de uma struct auxiliar que possui as chaves respectivas:

{{< highlight go "linenos=table" >}}
type Country struct {
    Location string
    Goods    []FoodPerDay
}

type FoodPerDay struct {
    Day  string
    Food string
}
{{< / highlight >}}

Sabemos que nosso json possui um array `Goods` de objetos `FoodPerDay` que contém as chaves `Day` e `Food` respectivamente.

E o decoding funciona da mesma forma:

{{< highlight go "linenos=table" >}}
package main

import (
    "encoding/json"
    "fmt"
)

var (
    jsonPerson  = `{"name": "Rodrigo", "age": 28}`
    jsonTech    = `{"project": "foo/bar", "stack": ["go", "elk", "elixir"]}`
    jsonCountry = `{
        "location": "Brazil", 
        "goods" :[
            {"day": "Wednesday", "food": "Feijoada e caipirinha"},
            {"day": "Monday", "food": "Virado à paulista"},
            {"day": "Friday", "food": "Arroz com molho de camarão e Merluza"}
        ]
    }`
)

type Person struct {
    Name string
    Age  int
}

type Tech struct {
    Project string
    Stack   []string
}

type Country struct {
    Location string
    Goods    []FoodPerDay
}

type FoodPerDay struct {
    Day  string
    Food string
}

func main() {
    // ==== Person ====
    var person Person
    json.Unmarshal([]byte(jsonPerson), &person)
    fmt.Printf("[Person] Name: %s - Age: %d\n", person.Name, person.Age)

    // ==== Tech ====
    var tech Tech
    json.Unmarshal([]byte(jsonTech), &tech)
    fmt.Printf("[Tech] Project: %s - Stack: %s\n", tech.Project, tech.Stack)

    // ==== Country ====
    var country Country
    json.Unmarshal([]byte(jsonCountry), &country)
    fmt.Printf("[Country] Location: %s - Goods: %s\n", country.Location, country.Goods)
}
{{< / highlight >}}

Resultado:

```text
girorme@DESKTOP-MN7HFPO ~/repositorios/go/jsonbyexample $ go run .
[Person] Name: Rodrigo - Age: 28
[Tech] Project: foo/bar - Stack: [go elk elixir]
[Country] Location: Brazil - Goods: [{Wednesday Feijoada e caipirinha} {Monday Virado à paulista} {Friday Arroz com molho de camarão e Merluza}]
```
## Encoding Json (Marshal)

Agora vamos ao encoding de json. Vamos separar o arquivo em duas funções: `decode()` e `encode()` para melhorar visualização:

{{< highlight go "linenos=table" >}}
package main

import (
    "encoding/json"
    "fmt"
)

var (
    jsonPerson  = `{"name": "Rodrigo", "age": 28}`
    jsonTech    = `{"project": "foo/bar", "stack": ["go", "elk", "elixir"]}`
    jsonCountry = `{
        "location": "Brazil", 
        "goods" :[
            {"day": "Wednesday", "food": "Feijoada e caipirinha"},
            {"day": "Monday", "food": "Virado à paulista"},
            {"day": "Friday", "food": "Arroz com molho de camarão e Merluza"}
        ]
    }`
)

type Person struct {
    Name string
    Age  int
}

type Tech struct {
    Project string
    Stack   []string
}

type Country struct {
    Location string
    Goods    []FoodPerDay
}

type FoodPerDay struct {
    Day  string
    Food string
}

func main() {
    decode()
    encode()
}

func decode() {
    // ==== Person ====
    var person Person
    json.Unmarshal([]byte(jsonPerson), &person)
    fmt.Printf("[Person] Name: %s - Age: %d\n", person.Name, person.Age)

    // ==== Tech ====
    var tech Tech
    json.Unmarshal([]byte(jsonTech), &tech)
    fmt.Printf("[Tech] Project: %s - Stack: %s\n", tech.Project, tech.Stack)

    // ==== Country ====
    var country Country
    json.Unmarshal([]byte(jsonCountry), &country)
    fmt.Printf("[Country] Location: %s - Goods: %s\n", country.Location, country.Goods)
}

func encode() {

}
{{< / highlight >}}

Similar ao decoding iremos nos basear nas estruturas que já possuímos. Agora podemos utilizar a função `json.Marshal`

{{< highlight go "linenos=table" >}}
func encode() {
    // ==== Person ====
    rodrigo := &Person{
        Name: "Rodrigo",
        Age:  28,
    }

    dataPerson, _ := json.Marshal(rodrigo)
    fmt.Println(string(dataPerson))

    // ==== Tech ====
    myStack := &Tech{
        Project: "Foo/Bar",
        Stack:   []string{"Elk", "Elixir", "Go"},
    }

    dataStack, _ := json.Marshal(myStack)
    fmt.Println(string(dataStack))

    // ==== Country ====
    brazil := &Country{
        Location: "Brazil",
        Goods: []FoodPerDay{
            {
                Day:  "Wednesday",
                Food: "Feijoada e caipirinha",
            },
            {
                Day:  "Monday",
                Food: "Virado à paulista",
            },
            {
                Day:  "Friday",
                Food: "Arroz com molho de camarão e Merluza",
            },
        },
    }

    dataCountry, _ := json.Marshal(brazil)
    fmt.Println(string(dataCountry))
}
{{< / highlight >}}

Resultado:

```text
girorme@machine ~/repositorios/go/jsonbyexample $ go run .
==== Decode ====
[Person] Name: Rodrigo - Age: 28
[Tech] Project: foo/bar - Stack: [go elk elixir]
[Country] Location: Brazil - Goods: [{Wednesday Feijoada e caipirinha} {Monday Virado à paulista} {Friday Arroz com molho de camarão e Merluza}]
==== Encode ====
{"Name":"Rodrigo","Age":28}
{"Project":"Foo/Bar","Stack":["Elk","Elixir","Go"]}
{"Location":"Brazil","Goods":[{"Day":"Wednesday","Food":"Feijoada e caipirinha"},{"Day":"Monday","Food":"Virado à paulista"},{"Day":"Friday","Food":"Arroz com molho de camarão e Merluza"}]}
```

### Observações

- A função `json.Marshal` retorna `[]byte` (array de byte) então precisamos converter o output para string:

{{< highlight go "linenos=table" >}}
dataStack, _ := json.Marshal(myStack)
fmt.Println(string(dataStack))
{{< / highlight >}}

- É necessário passar a referência da struct que será preenchida, por isso a utilização de `&` no inicio da struct que está sendo construída:

{{< highlight go "linenos=table" >}}
myStack := &Tech{
    Project: "Foo/Bar",
    Stack:   []string{"Elk", "Elixir", "Go"},
}
{{< / highlight >}}

- Para "pretty prints" podemos utilizar a função `json.MarshalIndent` Veja o exemplo no json de country:

{{< highlight go "linenos=table" >}}
dataCountry, _ := json.MarshalIndent(brazil, "", "  ")
fmt.Println(string(dataCountry))
{{< / highlight >}}

resultado:

```json
{
  "Location": "Brazil",
  "Goods": [
    {
      "Day": "Wednesday",
      "Food": "Feijoada e caipirinha"
    },
    {
      "Day": "Monday",
      "Food": "Virado à paulista"
    },
    {
      "Day": "Friday",
      "Food": "Arroz com molho de camarão e Merluza"
    }
  ]
}
```

## Struct tags
Podemos também utilizar tags de structs para aperfeiçoar a exibição/parsing de json. Muitas bibliotecas fazem o uso extensivo da feature para consolidar comportamentos diversos.

Para json podemos usar a tag: `json:field,options`

Com as tags podemos: mapear campos de struct para campos específicos do json, exibir um campo com nome diferente e ainda definir alguns comportamentos como omissão de chave/valor, considere o exemplo abaixo:

{{< highlight go "linenos=table" >}}
type Tech struct {
    Project string `json:"projeto"`
    Stack   []string
}

myStack := &Tech{
    Project: "Foo/Bar",
    Stack:   []string{"Elk", "Elixir", "Go"},
}

dataStack, _ := json.Marshal(myStack)
	fmt.Println(string(dataStack))
{{< / highlight >}}

a expressão acima faz tanto com que o output do campo Project seja exibido como projeto ou até msm o mapeamento inverso, onde o campo projeto irá ser exibido no campo food:

```json
{"projeto":"Foo/Bar","Stack":["Elk","Elixir","Go"]}
```

E o contrário:

{{< highlight go "linenos=table" >}}
var jsonTech = `{"projeto": "foo/bar", "stack": ["go", "elk", "elixir"]}`

type Tech struct {
    Project string `json:"projeto"`
    Stack   []string
}

var tech Tech
json.Unmarshal([]byte(jsonTech), &tech)
fmt.Printf("[Tech] Project: %s - Stack: %s\n", tech.Project, tech.Stack)
{{< / highlight >}}

```
[Tech] Project: foo/bar - Stack: [go elk elixir]
```

Fora o mapeamento básico temos algumas opções interessantes como:

- `omitempty`: Que não exibi saída desse campo quando ele estiver definido para o valor zero

- `-` : Que não traz o campo

ex:

{{< highlight go "linenos=table" >}}
type Person struct {
    Name string `json:"projeto,omitempty"`
    Password string `json:"-"`
}
{{< / highlight >}}

Para mais informações:

- https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go-pt

## Considerações

Existem casos onde não sabemos o json que está chegando e não queremos mapear isso via struct, para isso podemos resolver de algumas formas utilizando mapping para `map[T]interface{}` ou até msm com a estrutura `json.RawMessage` mas prefiro deixar para outro artigo já que esse já está um pouco "pesado" para quem está iniciando em GO xD

## Ref

- https://www.sohamkamani.com/golang/parsing-json/
- https://blog.golang.org/json
- https://gobyexample.com/json