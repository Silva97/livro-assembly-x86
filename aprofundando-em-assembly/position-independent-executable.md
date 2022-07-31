---
description: Explicando PIE e ASLR
---

# Position-independent executable

Como vimos no tópico [Endereçamento](../a-base/enderecamento.md) o processador calcula o endereço dos operandos na memória onde o resultado do cálculo será o endereço absoluto onde o operando está.

O problema disso é que o código que escrevemos precisa sempre ser carregado no mesmo endereço senão os endereços nas instruções estarão errados. Esse problema foi abordado no [tópico sobre MS-DOS](programando-no-ms-dos.md), onde a diretiva `org 0x100` precisa ser usada para que o NASM calcule o _offset_ correto dos símbolos senão os endereços estarão errados e o programa não funcionará corretamente.

Sistemas operacionais modernos têm um recurso de segurança chamado [ASLR](https://pt.wikipedia.org/wiki/Address\_space\_layout\_randomization) que dificulta a exploração de falhas de segurança no binário. Resumidamente ele carrega os endereços dos segmentos do executável em endereços aleatórios ao invés de sempre no mesmo endereço. Com o ASLR desligado os segmentos sempre são mapeados nos mesmos endereços.

Porém um código que acessa endereços absolutos jamais funcionaria apropriadamente com o ASLR ligado. É aí que entra o conceito de _Position-independent executable_ (PIE) que nada mais é que um executável com código que somente acessa endereços relativos, ou seja, não importa em qual endereço (posição) você carregue o código do executável ele irá funcionar corretamente.

{% hint style="info" %}
Na nossa PoC eu instruí para compilar o programa usando a _flag_ `-no-pie` no GCC para garantir que o _linker_ não iria produzir um executável PIE já que ainda não havíamos aprendido sobre o assunto. Mas depois de aprender a escrever código com endereçamento relativo em Assembly fique à vontade para remover essa _flag_ e começar a escrever programas independentes de posição.
{% endhint %}

## PIE em x86-64

Já vimos no tópico [Endereçamento](../a-base/enderecamento.md#enderecamento-em-x-86-64) que em x86-64 se tem um novo endereçamento relativo à RIP. É muito mais simples escrever código independente de posição no modo de 64-bit devido a isso.

Podemos usar a palavra-chave `rel` no endereçamento para dizer para o NASM que queremos que ele acesse um endereço relativo à RIP. Conforme exemplo:

```nasm
mov rax, [rel my_var]
```

Também podemos usar a diretiva `default rel` para que o NASM compile todos os endereçamentos como relativos por padrão. Caso você defina o padrão como endereço relativo a palavra-chave `abs` pode ser usada da mesma maneira que a palavra-chave `rel` porém para definir o endereçamento como absoluto.

Um exemplo de PIE em modo de 64-bit:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

char *assembly(void);

int main(void)
{
  printf("Resultado: %s\n", assembly());
  return 0;
}
```
{% endtab %}

{% tab title="assembly.asm" %}
```nasm
bits 64
default rel

section .rodata
    msg: db "Hello World!", 0

section .text

global assembly
assembly:
    lea rax, [msg]
    ret
```
{% endtab %}
{% endtabs %}

Experimente compilar sem a _flag_ `-no-pie` para o GCC na hora de _linkar_:

```
$ nasm assembly.asm -o assembly.o -felf64
$ gcc main.c -c -o main.o
$ gcc *.o -o test
```

Deveria funcionar normalmente. Mas experimente comentar a diretiva `default rel` na linha **2** e compilar novamente, você vai obter um erro parecido com esse:

![](<../.gitbook/assets/image (9).png>)

Repare que o erro foi emitido pelo _linker_ (`ld`) e não pelo compilador em si. Acontece que como usamos um endereço absoluto o NASM colocou o endereço do símbolo `msg` na _relocation table_ para ser resolvido pelo _linker_, onde o _linker_ é quem definiria o endereço absoluto do mesmo.

Só que como removemos o `-no-pie` o _linker_ tentou produzir um PIE e por isso emitiu um erro avisando que aquela referência para um endereço absoluto não pode ser usada.

## PIE em IA-32

Como o endereço relativo ao _Instruction Pointer_ só existe em modo de 64-bit, nos outros modos de processamento não é nativamente possível obter um endereçamento relativo. O compilador GCC resolve esse problema criando um pequeno procedimento cujo o único intuito é obter o valor no topo da pilha e armazenar em um registrador. Conforme ilustração abaixo:

```nasm
funcao:
    call __x86.get_pc_thunk.bx
    add ebx, 12345  ; Soma EBX com o endereço relativo 12345
    ; ...

__x86.get_pc_thunk.bx:
    mov ebx, [esp]
    ret
```

Ao chamar o procedimento `__x86.get_pc_thunk.bx` o endereço da instrução seguinte na memória é empilhado pela instrução [CALL](call-e-ret.md), portanto `mov ebx, [esp]` salva o endereço que EIP terá quando o procedimento retornar em EBX.

Quando a instrução `add ebx, 12345` é executada o valor de `EBX` coincide com o endereço da própria instrução ADD.
