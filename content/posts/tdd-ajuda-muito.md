+++
author = "girorme"
title = "TDD Ajuda muito!"
date = "2024-08-19"
description = "TDD ajuda muito e eu te mostro o porque!"
tags = [
    "tdd",
    "programming"
]
categories = [
    "tdd",
    "programming"
]
+++

TDD (Desenvolvimento Orientado por Testes) não só melhora a qualidade do código,
mas também incentiva práticas que mantêm o desenvolvimento ágil e antecipam problemas, 
resultando em um software mais robusto e fácil de manter

![tdd-ajuda-muito-1](/images/tdd-ajuda-muito-1.jpg)

Eai você ta de que lado? Ama ou odeia testes? :smiley:

# Agenda

1. **Introdução ao TDD**
   - O que é Test-Driven Development (TDD)?

2. **Um, de vários benefícios do TDD**

3. **TDD na Prática**
   - Exemplo de uso e implementação

4. **Conclusão**
   - Considerações finais sobre a adoção do TDD


## Introdução ao TDD

#### O que é Test-Driven Development (TDD)?

> Sem filosofar: No TDD você escreve o teste antes da implementação da fato.

E se você acha estranho ou simplesmente tem raiva ahuauhauh, é porque ainda não usufruiu de todos os benefícios que a prática nos proporciona como ~implementadores de bugs~ devs.

A ideia por mais "estranha" que pareça, funciona... e tem uma imagem famosa pra explicar, que tem em todo canto inclusive (eu fiz a minha no [excalidraw](https://excalidraw.com/)):

![tdd-ajuda-muito-2](/images/tdd-ajuda-muito-2.png)

Logo logo, aqui no artigo vamos ver a metodologia na prática.
Apesar de legal não vamos falar da história do tdd mas vale a pena entender as motivações:

> https://pt.wikipedia.org/wiki/Test-driven_development

Em resumo a técnica nasce de uma reflexão: Se testes são tão bons e proporcionam qualidade ao código, porque não estrapolar e escrever os testes antes mesmo do código?

## Um, de vários benefícios do TDD

> A capacidade de expressar a ideia de implementação no teste. Quando escrevemos um teste antes do código, orquestramos a ideia principal. A gente testa da forma que gostaríamos de usar a "api" desse código.

Por mais que você aplique vários padrões, uma hora ou outra (as vezes muito mais que uma hora outra xD) não escrevemos o código na sua melhor forma. E é muito comum essas "falhas" de design serem detectadas durante a escrita de um teste.

Você já deve ter passado por isso... na hora de escrever um teste e tentar mockar alguma resposta, sentiu dificuldade em deixar o comportamento flexível, fácil de testar. Geralmente o resultado dessa dificuldade é refatorar o código (ou simplesmente criar uma tarefa no backlog :p). Você poderia diminuir muito essa sensação adotando TDD...

### TDD na Prática

Por design o tdd nos obriga a criar implementações simples para que o teste passe. Então desde o inicio estamos refatorando o código.

O que pode parecer uma curva a mais de aprendizado (existe uma curva natural mas que após vencida traz maturidade incrível para a sua base de código), na verdade é a história do seu código sendo contada.

#### Exemplo de uso e implementação

Imagine o seguinte requisito:
```
Como jogador
Eu espero um jogo clássico simples de luta
Dessa forma posso escolher um personagem com poderes X
```

Poderíamos ter também critérios de aceite parecidos com:
```
- Quando eu selecionar um personagem, devo escolher seus poderes
- Após escolher os poderes, devo iniciar o jogo com o personagem escolhido
```

Existem vários formas de implementar as funcionalidades descritas. Utilizando tdd, poderíamos simplesmente começar com o seguinte trecho de testes (*O teste poderia ser muito menor mas vale a pena ver esse caso que a gente "injeta" o personagem no game*):

*pseudo-oop-code:*
```python
import my_test_framework

fun test_choose_character():
   game = new Game()

   character = new Character('Ryu')
   character->add_power('hadouken')

   game->add_character(character)
   
   assert game instanceof Game
   assert character instanceof Character
   assert character->get_powers() in ['hadouken']
   assert game->get_characters_count() == 1

   ...
```

Nós criamos quatro cenários de testes:

- Checamos se `new game()` de fato é uma instância válida de `Game`
- Checamos se `new character('Ryu')` de fato é uma instância válida de `Character`
- Checamos se o nosso personagem possui um poder chamado `hadouken`
- Checamos se temos 1 personagem dentro do jogo após adicionar o mesmo mais acima.

> Aqui independente da linguagem, caso você execute o teste, de cara, ele vai dar pau. Vai reclamar que `Game` e `Character` não existe e que nunca viu falar das funções/métodos `add_character`/`get_powers`/`get_characters_count`.

Essa á fase vermelha lá da primeira imagem...

De cara, aqui o benefício principal que comentei lá atrás: *A gente expressa como o código deve se comportar.*

Por mais que o código não exista, você já está expressando a ideia.

Dito isso, já da pra escrever as implementações na sua forma mais simples, apenas para os testes começarem a passar. Imagine que os números abaixo são etapas e a cada momento que eu executar novamente os testes, terei versões mais "maduras" e atendo as necessidades do teste etapa a etapa:

1.
```python
class Game:
   fun add_character(character):
      pass

   fun get_characters_count():
      return 1
```

*rodo o teste novamente, passa em partes, continua...*

2.
```python
class Character:
   fun add_power(power):
      pass

   fun get_powers():
      return ['hadouken']
```

*rodo o teste novamente, passa em partes, continua...*

Aqui por mais simples que seja, os testes passam e literalmente agora você precisa terminar de contar a história. Você vai implementar de fato parte a parte, rodar os testes e fazer com que ele chegue exatamente como planejou!

Interessante, certo? :)

### Dicas Rápidas para Começar

Aqui estão algumas dicas para iniciar com TDD:

- **Comece com testes pequenos e simples:** Escreva testes para as partes mais básicas e fáceis de implementar do seu código.
- **Teste uma funcionalidade por vez:** Foque em uma única função ou comportamento antes de seguir para a próxima.
- **Escreva testes que falhem primeiro:** Um teste que falha confirma que a funcionalidade ainda não foi implementada.
- **Refatore com frequência:** Após fazer o teste passar, revise o código para mantê-lo limpo e eficiente.
- **Use feedback rápido:** Execute os testes constantemente para obter feedback imediato sobre o progresso.
- **Não se preocupe em escrever o "código perfeito":** O código pode ser aprimorado durante a refatoração, o importante é garantir que ele funcione corretamente.
- **Tenha paciência:** O TDD pode parecer lento no início, mas a prática leva à eficiência.

### Conclusão
Para fechar... vale lembrar que o TDD, é mais do que uma técnica de desenvolvimento; é uma mudança de mentalidade. Ao adotar essa abordagem, você não está apenas escrevendo testes, você está moldando a maneira como seu código evolui, com mais confiança e menos surpresas desagradáveis. 

Então, se você ainda não deu uma chance ao TDD, já sabe...