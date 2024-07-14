+++
author = "Girorme"
title = "Aplicações desktop com GO e seu framework/lib front favorito"
date = "2021-04-03"
description = "Golang também é uma ótima opção para aplicações desktop"
tags = [
    "golang",
    "programming"
]
categories = [
    "golang",
    "react",
    "vue",
    "programming",
    "wails",
    "angular"
]
aliases = ["migrate-from-jekyl"]
+++

Sem dúvidas golang pode ser utilizado também para criar aplicações performáticas. Conheça o wails!

Já conhecemos go pela sua perfomance e produtividade. Temos também bibliotecas incríveis que são muito utilizadas diariamente. Uma que tem chamado minha atenção é a incrível [wails](https://wails.app/) que nos permite escrever aplicações desktop utilizando GO no backend e tecnologias web, como (React, Vue, Angular, etc) para prover o front. Isso nos da muitas possibilidades (o showcase inclusive é incrível ([wails-showcase](https://wails.app/showcase/))) e minha proposta é explorar o wails brevemente escrendo uma aplicação que faz scrap de links, então pegue sua cerveja!

<div style="text-align:center"><img width="500px" height="500px" src="/images/tech.png" /></div>

## Instalação wails

A instalação é simples e requer apenas alguns requisitos: [wails-getting-started](https://wails.app/gettingstarted/)

## Inicio projeto

Para inicializar um projeto wails é bem simples, seguido do comando: `wails init`, algumas informações são solicitadas e ao final escolhemos qual tecnologia usaremos no front, usarei Vue 3:

<div style="text-align:center"><img src="/images/go-wails-1.png" /></div>


## Conceitos importantes
O binding de dados e comunicação entre o frontend -> backend no wails são obtidos através de dois grandes atores: Bindings e Eventos.

### Bindings
No backend é possível bindar uma única função ou uma série de métodos de uma struct para que o frontend acesse facilmente esses métodos. O wails disponibiliza esses dados através de promises o que aumenta a facilidade de desenvolvimento / manutenção

### Eventos
O wails também da ao desenvolvedor uma forma muito simples de emitir eventos, tanto no backend quanto no frontend. Dessa forma é possível alcançar muitas features legais como notificações realtime (Juntando isso com um framework/lib como o vue ou react da pra fazer muita coisa mara!)

## Desenvolvendo a ferramenta
Vamos por etapas adicionando as dependências e codando features no backend e frontend

### Backend
O esqueleto do backend que é gerado é muito simples e tem apenas uma função bindada no arquivo `main.go`, vamos codar nosso scraper no arqivo `scraper.go`, antes disso irei instalar a lib [goquery](https://github.com/PuerkitoBio/goquery), que é um parser html muito simples de se utilizar:

{{< highlight bash "linenos=table" >}}
$ go get github.com/PuerkitoBio/goquery
{{< / highlight >}}

Feito isso podemos codar nossa funcionalidade de extrair links, abaixo deixo mais detalhes importantes sobre cada trecho

`scraper.go`

{{< highlight go "linenos=table" >}}
package main

import (
  "net/http"

  "github.com/PuerkitoBio/goquery"
  "github.com/wailsapp/wails"
)

type Scraper struct {
  log     *wails.CustomLogger
  runtime *wails.Runtime
}

func (s *Scraper) WailsInit(runtime *wails.Runtime) error {
  s.log = runtime.Log.New("Scraper")
  s.runtime = runtime
  return nil
}

func (s *Scraper) GetLinks(url string) (urls []string) {
  res, err := http.Get(url)

  if err != nil {
    s.runtime.Events.Emit("error", err.Error())
    s.log.Error(err.Error())
    return nil
  }

  defer res.Body.Close()

  doc, err := goquery.NewDocumentFromReader(res.Body)
  if err != nil {
    s.runtime.Events.Emit("error", err.Error())
    s.log.Error(err.Error())
    return nil
  }

  doc.Find("a").Each(func(i int, s *goquery.Selection) {
    if href, exists := s.Attr("href"); exists != false {
      urls = append(urls, href)
    }
  })

  return urls
}
{{< / highlight >}}

### Logs e Runtime

{{< highlight go "linenos=table" >}}
type Scraper struct {
  log     *wails.CustomLogger
  runtime *wails.Runtime
}

func (s *Scraper) WailsInit(runtime *wails.Runtime) error {
  s.log = runtime.Log.New("Scraper")
  s.runtime = runtime
  return nil
}
{{< / highlight >}}

Nos campos da struct `Scraper` utilizamos dois elementos importantes do Wails: `CustomLogger` e o `Runtime`, o primeiro nos permite emitir logs personalizados e o segundo nos da capacidade de emitir eventos com facilidade, como mencionado no inicio do artigo.

O método `WailsInit(runtime *wails.Runtime)` é nosso construtor que pode ser utilizado para algumas operações importantes antes de qualquer procedimento de fato nos métodos que são chamados pelo frontend.

Logo abaixo temos o método `GetLinks(url string)` que acessa a url informada e extrai alguns links. Nesse método temos algumas operações do wails acontecendo que são importantes de mencionar, ex:

{{< highlight go "linenos=table" >}}
if err != nil {
  s.runtime.Events.Emit("error", err.Error())
  s.log.Error(err.Error())
  return nil
}
{{< / highlight >}}

Quando temos um erro, emitimos o mesmo via o canal "error" e logamos o erro. Iremos escutar esse evento em breve no frontend. Emitir um evento do backend para o frontend é realmente simples!

Criado o arquivo podemos bindar o mesmo no arquivo `main.go`:

{{< highlight go "linenos=table" >}}
package main

import (
  "github.com/leaanthony/mewn"
  "github.com/wailsapp/wails"
)

func main() {
  js := mewn.String("./frontend/dist/app.js")
  css := mewn.String("./frontend/dist/app.css")
  scraper := &Scraper{}
  app := wails.CreateApp(&wails.AppConfig{
    Width:  1024,
    Height: 768,
    Title:  "scraper",
    JS:     js,
    CSS:    css,
    Colour: "#131313",
  })
  app.Bind(scraper)
  app.Run()
}
{{< / highlight >}}

### Frontend
Agora para o front, podemos fazer tudo o que fariamos em um projeto front normal: Instalar dependências, estilizar aplicação e no final das contas fazer a comunicação com o backend.

Para estilizar utilizei as deps `bootstrap` e `bootstrap-vue`, para isso basta acessar a pasta frontend e utilizar o já conhecido: `npm install x y z`.

Feito isso foi necessário uma modificação no arquivo `frontend/src/main.js`:

{{< highlight js "linenos=table" >}}
import 'core-js/stable';
import 'regenerator-runtime/runtime';
import Vue from 'vue';
import App from './App.vue';
import { BootstrapVue, IconsPlugin } from 'bootstrap-vue'
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'

Vue.config.productionTip = false;
Vue.config.devtools = true;
// Make BootstrapVue available throughout your project
Vue.use(BootstrapVue)
// Optionally install the BootstrapVue icon components plugin
Vue.use(IconsPlugin)

import * as Wails from '@wailsapp/runtime';

Wails.Init(() => {
  new Vue({
    render: h => h(App)
  }).$mount('#app');
});
{{< / highlight >}}

Adicionando as linhas:

{{< highlight js "linenos=table" >}}
import { BootstrapVue, IconsPlugin } from 'bootstrap-vue'
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'
Vue.use(BootstrapVue)
Vue.use(IconsPlugin)
{{< / highlight >}}

Feito isso criamos um componente para a feature `frontend/src/components/Scraper.vue`:

{{< highlight js "linenos=table" >}}
<template>
  <div>
    <h2>Link Scraper</h2>
    <div>
      <b-alert
      :show="dismissCountDown"
      dismissible
      fade
      variant="danger"
      @dismissed="dismissCountDown=0"
      @dismiss-count-down="countDownChanged"
      >
      Error: {{ error }}. Closing in {{ dismissCountDown }} seconds...
      </b-alert>
    </div>
    <b-form @submit="onSubmit">
      <b-form-group
        id="input-group-1"
        label="Url:"
        label-for="input-1"
        description="Url to scrap links"
      >
        <b-form-input
          id="input-1"
          v-model="form.url"
          placeholder="Enter url"
          required
        ></b-form-input>
      </b-form-group>
      <b-button type="submit" variant="primary">Scraaap!</b-button>
    </b-form>
    <div v-if="working">
      <h4>Scraping... <b-spinner type="grow"></b-spinner></h4>
    </div>
    <b-card class="mt-3" header="Urls">
      <pre class="m-0">{{ result }}</pre>
    </b-card>
  </div>
</template>

<script>
  import Wails from '@wailsapp/runtime';
  export default {
    data() {
      return {
        form: {
          url: ''
        },
        result: [],
        error: '',
        dismissSecs: 5,
        dismissCountDown: 0,
        working: false
      }
    },
    methods: {
      onSubmit(event) {
        event.preventDefault()
        this.working = true
        window.backend.Scraper.GetLinks(this.form.url).then(res => {
          this.result = res
        }).finally(() => {
          this.working = false
        })
      },
      countDownChanged(dismissCountDown) {
        this.dismissCountDown = dismissCountDown
      },
      showAlert() {
        this.dismissCountDown = this.dismissSecs
      }
    },
    mounted: function() {
      Wails.Events.On("error", errorMsg => {
        this.error = errorMsg
        this.showAlert()
      });
    }
  }
</script>
{{< / highlight >}}

Um componente muito simples. Que resulta no seguinte:

<div style="text-align:center"><img src="/images/go-wails-2.png" /></div>

### Trechos importantes

1 - Ao clicar no botão scrap, fazemos a chamada para o backend, para que isso seja possível, precisamos apenas desse trecho:

{{< highlight js "linenos=table" >}}
onSubmit(event) {
    event.preventDefault()
    this.working = true
    window.backend.Scraper.GetLinks(this.form.url).then(res => {
        this.result = res
    }).finally(() => {
        this.working = false
    })
},
{{< / highlight >}}

O método é acessado via `window.backend.Struct.Method`, o retorno é obtido via promise. Ali é setado algumas variáveis para controle de loading, etc.

2 - Para receber os eventos de erro que podem acontecer no método `GetLinks` lá do backend, fazemos o seuginte:

{{< highlight js "linenos=table" >}}
mounted: function() {
    Wails.Events.On("error", errorMsg => {
        this.error = errorMsg
        this.showAlert()
    });
}
{{< / highlight >}}

Utilizamos a função `mounted` do vue para iniciar o componente escutando esses eventos e como tudo ocorre de forma async não corremos o perigo do freezing de GUI (Programadores java/c# sabem bem como é isso quando não abrimos aquela threadzinha separada da main hehe)

Feito isso podemos ir ao build!

## Build
Para compilar a aplicação é simples: `wails build -d`, feito isso será criado um binário em `build/nome-app`

## Demo
<div style="text-align:center"><img src="/images/video-scraper.gif" /></div>

## Conclusão
Deixo aqui o ponta pé para você iniciar seu próximo projeto em wails! Um mundo de possibilidade te aguarda! (:

Caso queira dar uma olhada no repo exemplo: https://github.com/girorme/scraper

## Créditos
- https://wails.app/
- https://www.avantica.com/blog/wails-building-desktop-apps-in-go
- https://medium.com/@pliutau/building-a-desktop-app-in-go-using-wails-756c1f31f75
