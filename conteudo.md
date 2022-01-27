---
description: Conteúdo que será apresentado neste livro
---

# Conteúdo

Neste livro você irá aprender Assembly da arquitetura **x86** e **x86-64** desde os conceitos base até conteúdo mais "avançado". Digo conceitos "base" e não "básico" porque infelizmente o termo "básico" é mal empregado na internet afora. As pessoas estão acostumadas a verem conteúdo básico como algo superficial quando na verdade é a parte mais importante para o aprendizado. É a partir dos conceitos básicos que nós conseguimos aprender todo o resto. Mas infelizmente devido ao mal uso do termo ele acabou sendo associado a uma enorme quantidade de conteúdo superficial encontrado na internet.

> **Significado de básico**:\
> Que serve como base; essencial, basilar.\
> O mais relevante ou importante de; fundamental. \~ [Dicio](https://www.dicio.com.br/basico/)

Portanto que fique de prévio aviso que o conteúdo básico apresentado aqui não deve ser pulado, ele é de **extrema importância** e não será superficial como muitas vezes é visto em outras fontes de conteúdo na internet.

### Pré-requisitos

Neste livro não será ensinado a programar em C e nem muito menos como elaborar algoritmos usando o paradigma imperativo (vulgo "lógica de programação"). É recomendável ter experiência razoável em alguma linguagem de programação e ser capaz de escrever um "Olá Mundo" em C além de ao menos saber usar funções. Bem como também é importante que saiba usar a linha de comando mas não se preocupe, todos os comandos serão ensinados na hora certa. E claro, não ensinarei sobre sistemas de numeração como [binário](https://pt.wikipedia.org/wiki/Sistema\_de\_numera%C3%A7%C3%A3o\_bin%C3%A1rio), [hexadecimal](https://pt.wikipedia.org/wiki/Sistema\_de\_numera%C3%A7%C3%A3o\_hexadecimal) ou [octal](https://pt.wikipedia.org/wiki/Octal).

Por fim e o mais importante: Você precisa de um computador da arquitetura x86 rodando um sistema operacional de **64-bit**. Mas não adianta apenas tê-lo, use-o para programar o que iremos aprender aqui.

{% hint style="info" %}
Caso queira aprender C, o Mente Binária tem um treinamento gratuito intitulado [Programação Moderna em C](https://www.mentebinaria.com.br/treinamentos/programa%C3%A7%C3%A3o-moderna-em-c/).
{% endhint %}

### Ferramentas necessárias

Todas as ferramentas que utilizaremos são **gratuitas**, de **código aberto** e com versão para Linux e Windows. Não se preocupe pois você não terá que desembolsar nada e nem baixar softwares em um site "alternativo". Não ensinarei como instalar as ferramentas, você pode facilmente encontrar tutoriais pesquisando no Google ou na própria documentação da ferramenta. De preferência já deixe todas as ferramentas instaladas e, caso use o Windows, não esqueça de setar a [variável de ambiente](https://pt.wikipedia.org/wiki/Vari%C3%A1vel\_de\_ambiente) PATH corretamente.

Eis a lista de ferramentas:

* [nasm](https://www.nasm.us) -- Assembler que usaremos para nossos códigos em Assembly
* [gcc](https://gcc.gnu.org) -- Compilador de C que usaremos e ensinarei o Inline Assembly
* [mingw-w64](https://mingw-w64.org) -- É o porte do GCC para o Windows, caso use Windows instale esse especificamente e não o MinGW do projeto original.
* [dosbox](https://www.dosbox.com) -- Emulador do MS-DOS e arquitetura x86
* [qemu](https://www.qemu.org) -- Emulador de várias arquiteturas diferentes, usaremos a i386
* É recomendável que tenha o [make](http://gnuwin32.sourceforge.net/packages/make.htm), porém é opcional.

Qualquer dúvida sugiro que acesse o [fórum do Mente Binária](https://www.mentebinaria.com.br/forums/) a fim de tirar dúvidas e ter acesso a outros conteúdos.
