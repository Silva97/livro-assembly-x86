---
description: Aprendendo a mesclar Assembly e C
---

# Programando junto com C

Se você leu o conteúdo do livro até aqui já tem uma boa base para entender como o Assembly x86 funciona e como usá-lo. Também já tem uma boa noção do que está fazendo, entende bem o que o assembler faz e o que ele está produzindo como saída, sabe como efetuar cálculos em paralelo usando SSE inclusive com valores de ponto flutuante.

Em outras palavras você já tem a base necessária para realmente entender como as coisas funcionam, não decoramos instruções aqui mas sim entendemos as coisas em seu âmago. Agora está na hora de dar um passo a frente e entender como usar Assembly de uma maneira útil no "mundo real", vamos aprender a usar C e Assembly juntos afim de escrever programas.

Já estamos fazendo isso desde o começo mas não entramos em muitos detalhes pois eu queria que inicialmente o foco fosse em entender como as coisas funcionam, essa é a parte legal :grin:.

### Ferramentas

Como já mencionado antes vamos usar o GCC para compilar nossos códigos em C. Mas diferente dos capítulos anteriores que usamos o NASM, neste aqui vamos usar o assembler GAS com sintaxe da AT\&T **porque** assim aprendemos a ler código nessa sintaxe e a usar o GAS ao mesmo tempo.

Por convenção a gente usa a extensão `.s` (ao invés de `.asm`) para código ASM com sintaxe da AT\&T, então é a extensão que irei usar daqui em diante para nomear os arquivos.

Assim como fizemos em [A base](../a-base/) aqui está um código de teste para garantir que o seu ambiente está correto:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

int assembly(void);

int main(void)
{
  printf("Resultado: %d\n", assembly());
  return 0;
}
```
{% endtab %}

{% tab title="assembly.s" %}
```
    .text
    .globl assembly
assembly:
    movl $777, %eax
    ret
```
{% endtab %}
{% endtabs %}

O nome do executável do GAS é **as** e quando você instala o GCC ele vem junto, então você já tem ele instalado aí. Já pode tentar compilar com:

```
$ as assembly.s -o assembly.o
$ gcc main.c -c -o main.o
$ gcc *.o -o test
```

Caso tenha algum problema e precise de ajuda, pode entrar no [fórum do Mente Binária](https://www.mentebinaria.com.br/forums/) e fazer uma pergunta.

### Vendo o código de saída do GCC

Ao usar o GCC é possível passar o parâmetro `-masm=intel` para que o compilador gere código Assembly na sintaxe da Intel, onde por padrão ele gera código na sintaxe da AT\&T. Você pode ver o código de saída da seguinte forma:

```
$ gcc main.c -o main.s -S -masm=intel -fno-asynchronous-unwind-tables
```

Onde a _flag_ `-S` faz com que o compilador apenas compile o código, sem produzir o arquivo objeto de saída e ao invés disso salvando o código em Assembly. Pode ser útil fazer isso para aprender mais sobre a sintaxe do GAS.

A _flag_ `-fno-asynchronous-unwind-tables` serve para desabilitar as [diretivas CFI](https://sourceware.org/binutils/docs/as/CFI-directives.html) e melhorar a leitura do código de saída. Essas diretivas servem para gerar informação útil para um depurador mas para fins de leitura do código não precisamos delas.

Você também pode habilitar as otimizações do GCC com a opção `-O2` assim o código de saída será otimizado. Pode ser interessante fazer isso para aprender alguns truques de otimização.
