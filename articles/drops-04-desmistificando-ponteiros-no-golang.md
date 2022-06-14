---
tags: [go]
title: Drops #04: Desmistificando ponteiros no Golang!
---

# Drops #04: Desmistificando ponteiros no Golang!

![cover](https://res.cloudinary.com/practicaldev/image/fetch/s--whOeCZBq--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3qhbcja9m6ba63y7h3lc.png)

## Introdução

E ae dev, tudo bem com você?

Agora que FINALMENTE finalizei o MBA, aproveitei o tempo livre para dedicá-lo aos estudos do [Golang](https://golang.org/).

Confesso que eu estou apaixonado pela linguagem - ainda dando os primeiros passos claro - mas creio que já dá pra compartilhar uma parada muito massa da linguagem (e como ela implementa) que são os ponteiros!

Bora pro post?

Ah! mas antes disso... Esse post faz parte de uma série de artigos "drops" que tenho aqui! Veja a lista:

- [Drops #01: Corrigindo vulnerabilidades em dependências com Yarn! (ou quase)](https://dev.to/wnqueiroz/drops-01-corrigindo-vulnerabilidades-em-dependencias-com-yarn-ou-quase-2e2p)
- [Drops #02: Como alterar o autor do commit depois do push](https://dev.to/wnqueiroz/drops-02-como-alterar-o-autor-do-commit-depois-do-push-2mgg)
- [Drops #03: Usando aliases para importação de módulos em TypeScript!](https://dev.to/wnqueiroz/drops-03-usando-aliases-para-importacao-de-modulos-em-typescript-1fe9)
- [Drops #04: Desmistificando ponteiros no Golang!](https://dev.to/wnqueiroz/drops-04-desmistificando-ponteiros-no-golang-3kj9)

---

## Afinal, o que é um ponteiro?

Um ponteiro (ou apontador) nada mais é do que uma variável que, ao invés de armazenar um valor (true, "hello world"), ela armazena um endereço que está alocado na memória.

## Memória

Vamos entender a imagem a seguir:

![Representação da Memória](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lett8q04nv0fmxt2nljq.png)

Basicamente, bem a grosso modo, a memória é constituída por elementos que armazenam informações.

O endereço é uma posição onde os dados serão colocados (geralmente expressos em números hexadecimais). Eles podem conter apenas uma única informação.

O dado, por sua vez, é a informação presente em cada posição na memória.

Quando criamos uma variável, ela recebe um endereçamento na memória e através dessa variável podemos armazenar valores que serão alocados nesse endereço:

![Criação de variável x memória](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kqmkyw516ell7v88kop3.png)

Aqui podemos ler o seguinte:

A variável `a` que possui o endereço de memória `0xc000192020` está armazenando o dado `10` na posição deste endereço.

## Ponteiros no Go

É muito simples definir um ponteiro, basta adicionar o `*` junto ao tipo do ponteiro que estamos criando:

```go
package main

import "fmt"

func main() {
	var p *int

	fmt.Println(p) // output: <nil>
}
```

> 💡 O tipo do ponteiro indica qual é o tipo de dado que esse ponteiro irá manipular. No caso acima, criamos um ponteiro que pode "apontar" para endereços na memória de variáveis que armazenam dados do tipo inteiro.

No Go temos o conceito de _zero value_ que, ao definir uma variável sem atribuir um valor para ela, o Go irá atribuir um valor padrão para essa variável. Cada tipo possui o seu valor padrão (`0` para `int`, `false` para `bool`...), no caso do nosso ponteiro, o _zero value_ será `nil` dado que não atribuímos nenhum valor para ele. Ou melhor dizendo: nenhum endereço!

_E como podemos definir um endereço para o ponteiro?_ 🤔

Primeiro, criaremos outra variável no nosso código, e então, vamos atribuir o endereço dela para o ponteiro dessa forma:

```go
package main

import "fmt"

func main() {
	var p *int

	i := 10

	p = &i // atribuindo o endereço de i para o ponteiro

	fmt.Println(p) // output: algo como 0xc000192020
}
```

> 💡 O `&` usado antes da variável, indica que queremos obter o endereço na memória daquela variável.

A representação gráfica do que fizemos aqui está na imagem a seguir:

![Atribuição de endereço](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9faqss4csmdj27fl2n0q.png)

> 💡 Note que o ponteiro que criamos, também possui o seu próprio endereço na memória!

_Beleza, entendi! Até aqui o ponteiro já tá "apontando" pra um endereço... mas como eu faço pra saber o que esse endereço aí tá armazenando?_ 🤔

Basta utilizar o operador `*` junto ao ponteiro (`*p`). Esse é o processo de [desreferenciar](https://en.wikipedia.org/wiki/Dereference_operator) o ponteiro para que, ao invés dele retornar o endereço armazenado, ele ir lá naquele endereço e a partir de lá retornar o valor:

```go
package main

import "fmt"

func main() {
	var p *int

	i := 10

	p = &i

	fmt.Println(p)  // output: 0xc000192020
	fmt.Println(*p) // output: 10
}
```

Uma vez que desreferenciamos o ponteiro, podemos manipular o dado que está armazenado naquele endereço:

```go
package main

import "fmt"

func main() {
	var p *int

	i := 10

	p = &i

	fmt.Println(p)  // output: 0xc000192020
	fmt.Println(*p) // output: 10

	*p = 20

	fmt.Println(i)  // output: 20
	fmt.Println(*p) // output: 20
}
```

A representação gráfica do que fizemos aqui está na imagem a seguir:

![Manipulando dado pelo ponteiro](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r7ndqq0217ebgz0lc5jb.png)

## Quando usar ponteiros?

No Go, por padrão, tudo é _Pass By Value_. Isso significa que, quando passamos uma variável como parâmetro de uma função, essa variável é "duplicada" na memória e o que fazemos dentro do escopo da função acontece **apenas no escopo da função**:

```go
package main

import "fmt"

func increment(a int) int {
	a++

	fmt.Println(a) // output: 11

	return a
}

func main() {
	x := 10

	increment(x)

	fmt.Println(x) // output: 10
}
```

Como é realizada uma "cópia" da variável na memória, isso tem um custo, que pode se tornar problemático quando lidamos com um grande volume de dados nessa variável. Para economizarmos essa operação, é possível trabalhar com _Pass By Reference_ com a ajuda de ponteiros:

```go
package main

import "fmt"

func increment(a *int) {
	*a++

	fmt.Println(*a) // output: 11
	fmt.Println(a)  // output: 0xc0000180b0
}

func main() {
	x := 10

	increment(&x)

	fmt.Println(x)  // output: 11
	fmt.Println(&x) // output: 0xc0000180b0
}
```

Perceba que a função `increment` não retorna mais o valor manipulado e que além disso, passou a exigir um ponteiro/endereço de memória como parâmetro (`a *int`).

Dessa maneira, o parâmetro da função `increment` não foi duplicado na memória e passamos a trabalhar com a referência da variável `x` (que está fora do escopo da função `increment`).

A grosso modo, podemos trabalhar com ponteiros uma vez que lidamos com um volume grande de dados ou simplesmente querer trabalhar com _Pass By Reference_ ao invés de _Pass By Value_.

É importante entender que existem pontos positivos e negativos ao trabalhar com ponteiros. Nós podemos e devemos usá-los mas precisamos ter certeza do que estamos fazendo.

Ou seja: depende! HAHA Cabe a você avaliar os melhores cenários.

## Finalizando...

Bem, é isso, por hoje, é só!

Quero te agradecer por chegar até aqui, e queria lhe pedir também para me encaminhar as suas dúvidas, comentários, críticas, correções ou sugestões sobre a publicação.

Deixe seu ❤️ se gostou ou um 🦄 se esse post te ajudou de alguma maneira! Não se esqueça de ver os posts anteriores e me siga para maaaais conteúdos.

Até!
