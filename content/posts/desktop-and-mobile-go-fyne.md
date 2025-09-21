+++
author = "girorme"
title = "Aplicações desktop e mobile com go utilizando Fyne!"
date = "2025-09-17"
description = "Existem diversas formas de criar um app desktop e o fyne é uma delas"
tags = [
    "gui",
    "programming",
    "golang",
    "mobile"
]
categories = [
    "gui",
    "programming",
    "golang",
    "mobile"
]
+++

Mesmo depois de anos codando para web, não consigo deixar de lado o amor que tenho pelas aplicações escritas para desktop e terminal xD.

Muito tempo atrás passei alguns dias decidindo qual linguagem seria a melhor para se dedicar e dominar de alguma forma: Javão ou C#? As ideias que eu tinha em mente eram exclusivamente voltadas para apps visuais, com progress bars, atualização em tempo real da interface e outras coisas que estavam muito em alta na época.

O mais engraçado foi o quão rápido bati a cara na porta com a interface travada quando precisei fazer algum processo que demorava mais que alguns segundos xD

A partir dai vi que minha vida não seria fácil e precisei estudar muitos conceitos que os veteranos já estavam acostumados a tempo!

![gui-app-freezing](/images/gui-app-freezing.png)

## Por que apps com interface?

É fácil notar que hoje em dia muitos desenvolvedores dão muito mais atenção para aplicações feitas para a web, como api's, interfaces com fronts modernos ~~e complexos~~. Eu mesmo sempre esqueço o mundo mágico que é codar aplicação para desktop. Ai vira e mexe eu lembro de aplicações como vscode, openlens, proxies e tantas outras, ai da aquela saudade :smiley:

### E o Golang?

Aqui no blog já escrevi sobre o [wails](https://wails.io/), que é uma escolha sensata pra esse domínio de apps com interface, o único detalhe é a dependência que temos com JS. Se vc tem facilidade com tecnologias de front (react, vue, angular, etc) o wails é excelente e vai te atender bem. Vejo o fyne como uma alternativa mais simples que não te prende a outras tecnologias, então dependendo do seu momento, um ou outro pode ser interessante.

### O que codar pra conhecer o "toolkit"

O fyne usa esse termo para se descrever, "toolkit", que significa que várias ferramentas estão disponíveis pra tornar o ciclo de desenvolvimento mais fluído.

A intro é excelente:

> An easy to learn toolkit for creating graphical apps for desktop, mobile and web. Code once and build native apps for all platforms and stores.

Pt-br

> Um kit de ferramentas fácil de aprender para criar aplicativos gráficos para desktop, dispositivos móveis e web. Programe uma vez e crie aplicativos nativos para todas as plataformas e "lojas".

Nesse artigo vamos construir um "baixador" de arquivos. Um bom usecase pra demonstrar coisas como: progressbar, file dialogs, atualização de interface e até o uso de libs (pra facilitar o download e obter informações, tipo velocidade de download). O resultado final deve ficar parecido com isso aqui:

![downloadit](/images/downloadit.png)

## Estrutura do projeto

Pra fins de demonstração, uma estrutura bem simples é suficiente:

```
.
├── cmd
│   └── godownloadit
│       ├── FyneApp.toml
│       ├── icon.png
│       └── main.go
├── go.mod
├── go.sum
```

Conforme desenrolamos os tópicos, entramos nos detalhes

### Pré reqs

Antes de começar a receita, vamos precisar do seguinte:

- golang >= 1.19
- um compilador C
- driver gráfico

Tudo isso a gente consegue seguindo o [getting started](https://docs.fyne.io/started/) dos caras que é simples.

## Inicio da aventura

De inicio, precisamos inicializar nosso projeto e adicionar o fyne como dependência

### bootstrap do projeto
```
$ mkdir downloadit
$ cd $_
$ go mod init github.com/girorme/downloadit <-- aqui vc usa seu github ou outro nome que fizer sentido
```

Inicializou o projeto? Agora só adicionar o fyne como dep:
```
$ go get fyne.io/fyne/v2@latest
$ go install fyne.io/tools/cmd/fyne@latest
```

### Como organizar o código?
 A estrutura que utilizei para o exemplo considera a pasta `cmd/` como entrypoint. Lá vamos deixar nosso arquivo `main.go`.

 #### Ta funcionando ai?
 
Para validar a instalação e entender a estrutura mais simples possível de um app fyne, podemos testar esse código que está presente na doc de "primeiro app":

{{< highlight go "linenos=table" >}}
package main

import (
    "fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/widget"
)

func main() {
	a := app.New()
	w := a.NewWindow("Hello World")

	w.SetContent(widget.NewLabel("Hello World!"))
	w.ShowAndRun()
}
{{< / highlight >}}

Pra executar a aplicação:

```
$ go run cmd/downloadit/main.go
```

output:

![fyne-hello-world](/images/fyne-hello-world.png)

---

Geralmente uma aplicação fyne como essa do código, pode ser representada da seguinte forma:

![fyne-common-arq](/images/fyne-common-arq.png)

- App principal
  - Uma ou mais janelas dentro do app
    - Widgets dentro das janelas
---

### Compondo a aplicação

Esse é um exemplo que provavelmente você poderia fazer de outras formas, segui uma organização simples mas que nos permite atualizar e executar ações de forma mais simples, vamos trecho a trecho:

{{< highlight go "linenos=table" >}}
package main

import (
	"fmt"

	"fyne.io/fyne/v2"
	"fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/container"
	"fyne.io/fyne/v2/dialog"
	"fyne.io/fyne/v2/widget"
	"github.com/melbahja/got"
)

var (
	DEFAULT_SIZE                 = fyne.NewSize(520, 175)
	OPENING_FILE_SIZE            = fyne.NewSize(620, 504)
	APP_LABEL                    = "Download it!"
	DEFAULT_DOWNLOAD_SPEED_LABEL = "Avg speed: 0"
	DOWNLOADING_LABEL            = "Downloading..."
)

type DownloadIt struct {
	app                fyne.App
	window             fyne.Window
	mainTitle          *widget.Label
	urlEntry           *widget.Entry
	pBar               *widget.ProgressBar
	downloadBtn        *widget.Button
	downloadSpeedLabel *widget.Label
}
{{< / highlight >}}

Como qualquer outra aplicação, quando estamos trabalhando com interfaces gráficas uma hora ou outra reutilizamos valores, deixei algumas constantes pra esse fim na linha `14~20`.

Logo abaixo criei uma struct para simplificar o acesso aos componentes da nossa aplicação, logo não precisamos ficar passando componentes como argumentos em funções a todo momento, assim diminuimos a complexidade:

{{< highlight go "linenos=table" >}}
type DownloadIt struct {
	app                fyne.App
	window             fyne.Window
	mainTitle          *widget.Label
	urlEntry           *widget.Entry
	pBar               *widget.ProgressBar
	downloadBtn        *widget.Button
	downloadSpeedLabel *widget.Label
}
{{< / highlight >}}

Uma breve explicação do porque de cada componente utilizado:

- `app - fyne.App` - O core da nossa aplicação e onde atachamos novos componentes (representado no desenho anterior)
- `window - fyne.Window` - Nessa janela iremos colocar nossos widgets. Aqui como em outras linguagens / frameworks, podemos definir o tamanho da janela, comportamentos e eventos
- `mainTitle - *widget.Label` - As labels no wine são utilizadas para textos, uma forma simples para strings que podem ser atualizadas de acordo com a operação do momento
- `urlEntry - *widget.Entry` - Aqui fazendo um de x para na web, esse seria o nosso input de texto, aqui vamos setar o arquivo que será baixado
- `pBar - *widget.ProgressBar` - ProgressBar que vamos utilizar para trackear o processo de download
- `downloadBtn - *widget.Button` - Os botões são botões xD, através deles conseguimos emitir eventos e nesse vamos iniciar o nosso download e atualizar alguns  estados
- `downloadSpeedLabel - *widget.Label` - Essa label será atualizada em tempo real com a velocidade atual de download

---

#### Esqueleto dos componentes

A função `main()` nessa aplicação servirá como entrypoint para o setup e execução do app:

{{< highlight go "linenos=table" >}}
func main() {
	app := NewApp()

	app.setupContent()
	app.run()
}
{{< / highlight >}}

Como boa prática em aplicações GO é recomendado e funciona muito bem uma função que irá inicializar nossa struct, aqui faremos através da função `NewApp()`:

{{< highlight go "linenos=table" >}}
func NewApp() *DownloadIt {
	app := app.New()
	window := app.NewWindow(APP_LABEL)
	mainTitle := widget.NewLabel(APP_LABEL)
	urlEntry := widget.NewEntry()
	urlEntry.SetPlaceHolder("Enter URL...")

	pBar := widget.NewProgressBar()
	pBar.Max = 99.99

	downloadSpeedLabel := widget.NewLabel(
        DEFAULT_DOWNLOAD_SPEED_LABEL
    )

	return &DownloadIt{
		app:                app,
		window:             window,
		mainTitle:          mainTitle,
		urlEntry:           urlEntry,
		pBar:               pBar,
		downloadSpeedLabel: downloadSpeedLabel,
	}
}
{{< / highlight >}}

Aqui nós criamos nossa primeira composição:

- Criamos um novo `fyne.App` através da função `app.New()`
- Atachamos a esse app uma janela com `app.NewWindow(APP_LABEL)`. Inclusive utilizando a constante definida anteriormente
- Criamos os componentes que serão utilizados e setamos alguns valores/propriedades:
    - Labels com o texto que precisamos
    - Entry e adicionamos um placeholder (legenda)
    - ProgressBar e definimos qual o valor máximo

No final, retornamos a referência a struct.

---

Uma vez com os componentes estabelecidos, agora vamos organiza-los no layout e tbm adicionar os comportamentos necessários para que cada um funcione da forma esperada

#### Setup dos componentes

Agora que temos a struct com os componentes disponíveis, vamos criar os métodos para essa struct. Começando por `setupContent()`:

{{< highlight go "linenos=table" >}}
func (app *DownloadIt) setupContent() {
	app.resizeWindow(DEFAULT_SIZE)
	app.setupDownload()
	app.window.SetContent(container.NewVBox(
		app.mainTitle,
		app.urlEntry,
		app.pBar,
		app.downloadSpeedLabel,
		app.downloadBtn,
	))
}
{{< / highlight >}}

O comportamento padrão de janela do fyne é ir crescendo conforme os componentes são adicionados. Para criar um tamanho pré-definido vamos usar a função `app.resizeWindow()`:

{{< highlight go "linenos=table" >}}
func (app *DownloadIt) resizeWindow(size fyne.Size) {
	app.window.Resize(fyne.NewSize(size.Width, size.Height))
}
{{< / highlight >}}

- Para facilitar as mudanças de tamanho quando necessário!

#### Lógica do download

Aqui nós criamos o botão e associamos uma função de evento para o click

{{< highlight go "linenos=table" >}}
func (app *DownloadIt) setupDownload() {
	app.downloadBtn = widget.NewButton("Download", func() {
		app.downloadFile()
	})
}
{{< / highlight >}}

Aqui alimentamos nossa struct com o botão de download definido anteriormente. Associamos a string "Download" a ele. Entrando um nível abaixo, vamos a função `app.downloadFile()` que é a principal do programa, aqui basicamente acontece o mais interessante:

{{< highlight go "linenos=table" >}}
func (app *DownloadIt) downloadFile() {
	if app.urlEntry.Text == "" {
		dialog.ShowInformation(
			"URL required", 
			"Please enter a valid URL to download.", 
			app.window
		)
		return
	}

	app.updateStartAndFinishedState(true)
	app.resizeWindow(OPENING_FILE_SIZE)

	dialog.ShowFileSave(
		func(writer fyne.URIWriteCloser, err error) 
	{
		defer app.resizeWindow(DEFAULT_SIZE)

		if err != nil || writer == nil {
			app.mainTitle.SetText(APP_LABEL)
			return
		}
		defer writer.Close()

		fileToSave := writer.URI().Path()
		fmt.Printf(
			"File to save: %s - %s\n", 
			app.urlEntry.Text, 
			fileToSave
		)
		
		downloadSpeedStr := DEFAULT_DOWNLOAD_SPEED_LABEL

		go func() {
			g := got.New()
			g.ProgressFunc = func(d *got.Download) {
				progress := 
					float64(d.Size()) / 
					float64(d.TotalSize()) * 100
				fyne.Do(func() {

					if d.Speed() > 0 {
						downloadSpeedStr = fmt.Sprintf(
							"Avg speed: %s", 
							formatDownloadSpeed(d.Speed())
						)
					}

					app.pBar.SetValue(progress)
					app.downloadSpeedLabel.SetText(
						downloadSpeedStr
					)
				})
			}

			err := g.Download(app.urlEntry.Text, fileToSave)
			if err != nil {
				dialog.ShowError(
					fmt.Errorf(
						"error downloading file: %s", 
						err
					), 
					
					app.window
				)
				app.updateStartAndFinishedState(false)
				return
			}

			app.updateStartAndFinishedState(false)
		}()
	}, app.window)
}
{{< / highlight >}}

Vamos por partes para não ficar massante:

#### Validação da url de download

{{< highlight go "linenos=table" >}}
if app.urlEntry.Text == "" {
	dialog.ShowInformation(
		"URL required", 
		"Please enter a valid URL to download.", 
		app.window
	)
	return
}
{{< / highlight >}}

Nosso programa tem um input da url, se tiver vazio utilizamos a função `dialog.ShowInformation()` para exibir um messageBox.

> **Repare aqui o benefício de ter o programa estruturado através de uma struct para acesso aos componentes. Temos acesso a tudo! Veja que passamos `app.window()` para a função de messagebox de uma forma muito simplificada!!!**

#### Atualização de estado no inicio e fim da operação de download

A função `app.updateStartAndFinishedState(bool)` que vem a seguir é um "helper" para atualizar algumas strings e comportamentos quando começamos e finalizamos um download, essa prática te ajuda a executar algumas tarefas repetitivas onde precisamos atualizar múltiplos componentes em um determinado momento:

{{< highlight go "linenos=table" >}}
func (app *DownloadIt) updateStartAndFinishedState(
	starting bool
) {
	if starting {
		app.mainTitle.SetText(DOWNLOADING_LABEL)
		app.updateDownloadBtnText(DOWNLOADING_LABEL, true)
		return
	}

	app.mainTitle.SetText(APP_LABEL)
	dialog.ShowInformation(
		"Download completed", 
		"File downloaded successfully!", 
		app.window
	)
	app.updateDownloadBtnText("Download", false)
	app.pBar.SetValue(0.0)
	app.downloadSpeedLabel.SetText(
		DEFAULT_DOWNLOAD_SPEED_LABEL
	)
}
{{< / highlight >}}

Aqui temos coisas que já vimos antes exceto uma nova forma de atualizar componentes, que está presente na função `updateStartAndFinishedState(bool)`:

{{< highlight go "linenos=table" >}}
func (app *DownloadIt) updateDownloadBtnText(
	text string, 
	disable bool
) {
	fyne.Do(func() {
		app.downloadBtn.SetText(text)

		if disable {
			app.downloadBtn.Disable()
			return
		}

		app.downloadBtn.Enable()
	})
}
{{< / highlight >}}

Essa função é chamada em um momento do nosso programa que a operação estará dentro uma go routine, então aqui o uso do `fyne.Do()` é necessária. Ela vai garantir que nossa ui seja atualizada de forma correta. Aqui uma explicação muito bacana gerada com LLM:


> “Pense na interface gráfica como um palco de teatro. Só um ator pode estar no palco por vez (a thread principal do aplicativo). Os ajudantes que trabalham nos bastidores (as goroutines) não podem simplesmente invadir o palco e mexer nos cenários (os widgets), porque isso causaria confusão e poderia quebrar a peça (erros e concorrência). Em vez disso, eles entregam um bilhete para o diretor de cena (fyne.Do). Esse diretor garante que as mudanças sejam feitas no momento certo e de forma organizada, mantendo o espetáculo fluindo sem problemas.”

Utilizei a "flag" `disable` nessa função para saber o momento que estamos fazendo o download e precisamos bloquear o botão, assim evitamos que o usuário clique mais de uma vez no mesmo. Sagaz e simples certo? xD

#### Alterando o tamanho da janela

Alteramos o tamanho da janela uma vez durante o setup da aplicação. Vamos precisar alterar novamente agora. O motivo: Vamos abrir um "File dialog" para selecionar o destinho do nosso download. Para que a nova janela fique em um tamanho adequado, mostrando as pastas e outros arquivos, vamos fazer esse resize.

#### Mostrando o "File Dialog"

Tudo que ficar englobado em `dialog.ShowFileSave()` acontecerá logo que a janela para seleção de arquivo aparecer e selecionarmos um arquivo ou cancelar a operação. Essa função nos da dois parametros

- `writer fyne.URIWriteCloser` - Aqui um handler apontando para o arquivo selecionado para o usuário, com esse write nós mandamos o dado de fato para dentro do arquivo
- `err error` - Que indica se tivemos algum problema na hora de criar / fazer overwrite do arquivo

De cara podemos fazer a validação em caso de erro / cancelamento:

{{< highlight go "linenos=table" >}}
defer app.resizeWindow(DEFAULT_SIZE)

if err != nil || writer == nil {
	app.mainTitle.SetText(APP_LABEL)
	return
}
defer writer.Close()
{{< / highlight >}}

Nesse caso a gente volta a janela ao tamanho padrão e setamos o texto para quando não há download em andamento. E o `writer.Close()` fecha o handle do arquivo ao término da função

#### Iniciando o download e obtendo informações do arquivo que está sendo baixado

A partir de agora nós setamos a label da velocidade do download (padrão sempre em 0 (zero)) e tbm vamos atualizar a label conforme a velocidade varia:

{{< highlight go "linenos=table" >}}
downloadSpeedStr := DEFAULT_DOWNLOAD_SPEED_LABEL

go func() {
	g := got.New()
	g.ProgressFunc = func(d *got.Download) {
		progress := 
			float64(d.Size()) / 
			float64(d.TotalSize()) * 100
		fyne.Do(func() {

			if d.Speed() > 0 {
				downloadSpeedStr = fmt.Sprintf(
					"Avg speed: %s", 
					formatDownloadSpeed(d.Speed())
				)
			}

			app.pBar.SetValue(progress)
			app.downloadSpeedLabel.SetText(
				downloadSpeedStr
			)
		})
	}

	...
{{< / highlight >}}

Os detalhes

- `g := got.New()` - Aqui estamos utilizando a lib [Got](https://github.com/melbahja/got) que nos permite baixar com facilidade arquivos. É possível utilizar o got como programa ou módulo. Aqui ela foi instalada da seguinte forma:
  - 