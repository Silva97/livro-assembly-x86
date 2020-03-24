---
description: Explicação do porque aprender Assembly é útil
---

# Introdução

Sempre recebi muitas perguntas do tipo "como aprender Assembly?". Já faz um tempo que tenho em mente a ideia de produzir um conteúdo que sirva de introdução neste mundo.  
Porém a ideia aqui não é servir apenas para ensinar a fazer um "Hello World" e te deixar sem entender coisa alguma do que está fazendo.  
Vamos abordar parte por parte dos conceitos da linguagem Assembly até entendê-la completamente, e para isso, vamos inclusive ver um pouco de código de máquina da arquitetura x86.

### Por que aprender Assembly?

Se você quer aprender Assembly achando que vai conseguir fazer programas mais rápidos diretamente em Assembly, saiba que está enganado. Os algoritmos de otimização dos compiladores de hoje em dia tem uma eficiência impressionante a ponto de superar as habilidades humanas. Ou seja, é muito mais eficiente escrever um código em C, por exemplo, do que diretamente em Assembly. Claro que ainda é possível fazer código mais eficiente diretamente em Assembly, porém apenas em casos específicos e se você tiver bastante experiência. Ou seja é mais interessante usar Assembly em conjunto com C do que fazer todo o software em Assembly.  
Aprender Assembly nos dias atuais tem sim utilidade e irei listar algumas:

1. Engenharia Reversa \(de softwares\) Entender como um software funciona na prática só é de fato possível se você souber Assembly. Usar [_decompiler_](https://pt.wikipedia.org/wiki/Descompilador) até funciona em parte, mas não pense que isso tem grandes utilidades o tempo todo.
2. Exploração de binários Para quem quer fazer testes de segurança em binários, Assembly é indispensável.
3. Otimização de código Sim, como eu já disse é mais eficiente não escrever o código diretamente em Assembly. Porém saber Assembly e usar esse conhecimento para estudar o código de saída de um compilador de uma determinada linguagem \(GCC por exemplo\) vai te fazer aprender muita coisa sobre o código resultante. Esse conhecimento vai te ajudar e muito a fazer códigos mais eficientes nesta linguagem. Isto é claro, é especialmente útil em linguagens que te permitem prever o código de saída. \(volto a repetir C como exemplo\)
4. Otimização de código² Também dá para fazer otimizações de código para processadores específicos manualmente. Podemos ver vários exemplos desta façanha no famigerado ffmpeg, [veja aqui](https://github.com/FFmpeg/FFmpeg/blob/a0ac49e38ee1d1011c394d7be67d0f08b2281526/libavcodec/x86/ac3dsp.asm).
5. Tarefas de "baixo nível" Algumas instruções específicas de determinados processadores são basicamente impossíveis de serem utilizadas em uma linguagem de alto nível. Uma tarefa básica em Assembly, como chamar uma interrupção de software, é impossível de ser feita em uma linguagem de alto nível como C.
6. C & Assembly Termino falando que há incrível vantagem de se saber Assembly quando se programa em C. Não só pelo [inline Assembly](https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html) mas também pela previsibilidade do código, que eu já mencionei. \(você vai entender o que é isso se continuar lendo o livro\)

Como podemos ver o conhecimento da linguagem Assembly nos dias atuais é mais útil do que a utilização dela em si.

### O que é Assembly?

Caso ainda não saiba, cada arquitetura de computador tem sua própria linguagem ASM \(Assembly\). A que nós estudaremos aqui é da [arquitetura x86](https://pt.wikipedia.org/wiki/X86).  
Já escrevi um artigo no Medium explicando de fato o que é a linguagem Assembly e o porque de cada arquitetura ter um ASM diferente.  
Para não repetir o conteúdo vou apenas deixar o link do artigo:

* [Entenda o que é Assembly](https://medium.com/@FreeDev/entenda-o-que-%C3%A9-assembly-ed64526cab49)

É muito importante a leitura deste artigo mesmo que você acredite já saber o que é Assembly. É um texto curto, não tem porque evitá-lo. E como eu disse no início do artigo:

> Saber o que é Assembly e entender o que é Assembly são duas coisas diferentes.

{% hint style="info" %}
Apesar de eu estar utilizando letra maiúscula para escrever a palavra "Assembly", ela na verdade não é um substantivo próprio mas sim a palavra "montagem" em inglês.  
Estou fazendo isso meramente por uma estilização pessoal.
{% endhint %}

### O que é assembler?

O assembler é o software encarregado de converter o código em Assembly para o código de máquina. Muito antigamente, nos primórdios da computação, uma pessoa manualmente convertia os códigos em Assembly para código de máquina. Depois inventaram uma máquina para automatizar esta tarefa, sendo chamada de "assembler". Somente nos dias atuais que o "assembler" é um software de computador.  
Geralmente o processo de converter Assembly para código de máquina é chamado de "montar", ou seja, o assembler "monta" o código.

{% hint style="warning" %}
É muito comum as pessoas se confundirem e dizerem "linguagem _assembler_", o que está errado. Cuidado para não se confundir também.
{% endhint %}

### O autor

Meu nome é Luiz Felipe. Pronto, agora você sabe tudo sobre mim.

* [GitHub](https://github.com/Silva97)
* [Facebook](https://www.facebook.com/B4.0E.B0.48.CD.10.B0.69.CD.10.C3)
* [Medium](https://medium.com/@FreeDev)

### Licença

Este conteúdo está sendo compartilhado sobre os termos da licença [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/legalcode) e é totalmente de minha autoria.  
Esta licença permite que você compartilhe o conteúdo, mesmo que para fins lucrativos, desde que seja compartilhado sob os termos da mesma licença e sem adicionar novas restrições. Como também é necessário que me dê os créditos pelo trabalho original.

Quando for me dar os créditos, basta citar meu nome e deixar o link para o meu Facebook e/ou GitHub.  
Outra opção é que deixe um link para este material original além de citar meu nome.

Para mais detalhes da licença, pode consultar no link abaixo:

* [Atribuição-CompartilhaIgual 3.0 Não Adaptada \(CC BY-SA 3.0\)](https://creativecommons.org/licenses/by-sa/3.0/deed.pt)

