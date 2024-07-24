+++
author = "girorme"
title = "Desktop Apps with GO and Your Favorite Frontend Framework/Lib"
date = "2021-04-03"
description = "Golang is also a great option for desktop applications"
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
+++

No doubt, golang can be used to create high-performance applications. One interesting thing is developing desktop apps with this language.

We already know Go for its performance and productivity. We also have some incredible libraries that are used daily. One that has caught my attention is the amazing [Wails](https://wails.app/), which allows us to write desktop applications using GO on the backend and web technologies (like React, Vue, Angular, etc.) for the front end. This opens up many possibilities (the showcase is incredible, check it out: [wails-showcase](https://wails.app/showcase/)), and my proposal is to briefly explore Wails by writing an app that scrapes links. So, grab your beer!

<div style="text-align:center"><img width="500px" height="500px" src="/images/tech.png" /></div>

## Wails Installation

Installation is simple and just requires a few prerequisites: [wails-getting-started](https://wails.app/gettingstarted/)

## Starting a Project

Starting a Wails project is super simple. Just run the command `wails init`, answer a few questions, and at the end, choose which technology you'll use for the front end. I'll use Vue 3:

<div style="text-align:center"><img src="/images/go-wails-1.png" /></div>

## Important Concepts

Data binding and communication between the frontend and backend in Wails are achieved through two main actors: Bindings and Events.

### Bindings

In the backend, you can bind a single function or a series of methods from a struct so that the frontend can easily access these methods. Wails makes these data available through promises, which simplifies development and maintenance.

### Events

Wails also gives developers a very simple way to emit events, both on the backend and frontend. This way, you can achieve many cool features like real-time notifications (Combining this with a framework/lib like Vue or React, you can do a lot of awesome stuff!)

## Developing the Tool

Let's go step by step, adding dependencies and coding features in the backend and frontend.

### Backend

The backend skeleton that's generated is very simple and has only one bound function in the `main.go` file. Let's code our scraper in the `scraper.go` file. Before that, I'll install the [goquery](https://github.com/PuerkitoBio/goquery) lib, which is a very simple HTML parser to use:

```bash
$ go get github.com/PuerkitoBio/goquery
```

Once that's done, we can code our functionality to extract links. Below are some important details about each snippet.

`scraper.go`

```go
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
    if href, exists := s.Attr("href"); exists {
      urls = append(urls, href)
    }
  })
  return urls
}
```

### Logs and Runtime

```go
type Scraper struct {
  log     *wails.CustomLogger
  runtime *wails.Runtime
}

func (s *Scraper) WailsInit(runtime *wails.Runtime) error {
  s.log = runtime.Log.New("Scraper")
  s.runtime = runtime
  return nil
}
```

In the `Scraper` struct fields, we use two important elements from Wails: `CustomLogger` and `Runtime`. The first allows us to emit custom logs, and the second gives us the ability to emit events easily, as mentioned at the beginning of the article.

The `WailsInit(runtime *wails.Runtime)` method is our constructor that can be used for some important operations before any actual procedures in the methods called by the frontend.

Right below, we have the `GetLinks(url string)` method that accesses the provided URL and extracts some links. In this method, we have some Wails operations happening that are important to mention, e.g.:

```go
if err != nil {
  s.runtime.Events.Emit("error", err.Error())
  s.log.Error(err.Error())
  return nil
}
```

When we have an error, we emit it via the "error" channel and log the error. We'll listen to this event soon on the frontend. Emitting an event from the backend to the frontend is really simple!

Once the file is created, we can bind it in the `main.go` file:

```go
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
```

### Frontend

Now for the front, we can do everything we would do in a normal front project: Install dependencies, style the app, and in the end, make the communication with the backend.

To style, I used the `bootstrap` and `bootstrap-vue` dependencies. For that, just access the frontend folder and use the well-known: `npm install x y z`.

Once that's done, a modification is needed in the `frontend/src/main.js` file:

```js
import 'core-js/stable';
import 'regenerator-runtime/runtime';
import Vue from 'vue';
import App from './App.vue';
import { BootstrapVue, IconsPlugin } from 'bootstrap-vue';
import 'bootstrap/dist/css/bootstrap.css';
import 'bootstrap-vue/dist/bootstrap-vue.css';

Vue.config.productionTip = false;
Vue.config.devtools = true;
// Make BootstrapVue available throughout your project
Vue.use(BootstrapVue);
// Optionally install the BootstrapVue icon components plugin
Vue.use(IconsPlugin);

import * as Wails from '@wailsapp/runtime';

Wails.Init(() => {
  new Vue({
    render: h => h(App)
  }).$mount('#app');
});
```

Adding the lines:

```js
import { BootstrapVue, IconsPlugin } from 'bootstrap-vue';
import 'bootstrap/dist/css/bootstrap.css';
import 'bootstrap-vue/dist/bootstrap-vue.css';
Vue.use(BootstrapVue);
Vue.use(IconsPlugin);
```

Once that's done, we create a component for the feature `frontend/src/components/Scraper.vue`:

```js
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
        window.backend.Scraper.GetLinks

(this.form.url).then(res => {
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
```

A very simple component that results in the following:

<div style="text-align:center"><img src="/images/go-wails-2.png" /></div>

### Important Snippets

1 - When clicking the scrap button, we make a call to the backend. To make this possible, we only need this snippet:

```js
onSubmit(event) {
    event.preventDefault();
    this.working = true;
    window.backend.Scraper.GetLinks(this.form.url).then(res => {
        this.result = res;
    }).finally(() => {
        this.working = false;
    });
},
```

The method is accessed via `window.backend.Struct.Method`, and the return is obtained via promise. Some variables are set there to control loading, etc.

2 - To receive the error events that might happen in the `GetLinks` method from the backend, we do the following:

```js
mounted: function() {
    Wails.Events.On("error", errorMsg => {
        this.error = errorMsg;
        this.showAlert();
    });
}
```

We use the Vue `mounted` function to start the component listening to these events, and since everything happens asynchronously, we don't risk GUI freezing (Java/C# programmers know this well when we don't open that little separate thread from the main one, hehe).

Once that's done, we can build!

## Build

To compile the app, it's simple: `wails build -d`, and a binary will be created in `build/nome-app`.

## Demo

![video-scraper](/images/video-scraper.gif)

## Conclusion

Here's the kick-off for you to start your next project in Wails! A world of possibilities awaits you! (:

If you want to check out the example repo: https://github.com/girorme/scraper

## Credits

- https://wails.app/
- https://www.avantica.com/blog/wails-building-desktop-apps-in-go
- https://medium.com/@pliutau/building-a-desktop-app-in-go-using-wails-756c1f31f75