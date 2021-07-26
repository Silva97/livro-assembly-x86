---
description: Listando algumas instruções de movimentação de dados do SSE.
---

# Instruções de movimentação de dados

### MOVAP\(S\|D\)/MOVUP\(S\|D\) \| Move Aligned/Unaligned Packed \(Single\|Double\)-precision floating-point

```text
MOVAPS xmm(n), xmm(n)
MOVAPS xmm(n), float(4)
MOVAPS float(4), xmm(n)

MOVUPS xmm(n), xmm(n)
MOVUPS xmm(n), float(4)
MOVUPS float(4), xmm(n)


MOVAPD xmm(n), xmm(n)
MOVAPD xmm(n), double(2)
MOVAPD double(2), xmm(n)

MOVUPD xmm(n), xmm(n)
MOVUPD xmm(n), double(2)
MOVUPD double(2), xmm(n)
```

As instruções MOVAPS e MOVUPS fazem a mesma coisa: Movem 4 valores _float_ empacotados entre registradores XMM ou de/para memória principal. MOVAPD e MOVUPD porém lida com 2 valores _double_.

A diferença é que a instrução MOVAPS/MOVAPD espera que o endereço do valor na memória esteja alinhado a um valor de 16 bytes, caso não esteja a instrução dispara uma exceção \#GP \(General Protection ou "segmentation fault" como é conhecido no Linux\). O motivo dessa instrução exigir isso é que acessar o endereço alinhado é muito mais performático.

Já a instrução MOV**U**PS/MOV**U**PD pode acessar um endereço de memória desalinhado \(_**u**naligned_\) sem ocorrer nenhum erro, porém ela é menos performática.

Um exemplo de uso da MOVAPS na nossa PoC:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

void assembly(float *array);

int main(void)
{
  float array[4];
  assembly(array);

  printf("%f, %f, %f, %f\n", array[0], array[1], array[2], array[3]);
  return 0;
}
```
{% endtab %}

{% tab title="assembly.asm" %}
```
bits 64
default rel

section .rodata align=16
    local_array: dd 1.23
                 dd 2.45
                 dd 3.67
                 dd 4.89

section .text

global assembly
assembly:
    movaps xmm5, [local_array]
    movaps [rdi], xmm5
    ret

```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Sem entrar em detalhes ainda sobre a convenção de chamada, o ponteiro recebido como argumento pela função **assembly\(\)** está no registrador RDI.

Sobre o atributo **align=16** usado na seção **.rodata** ele serve para fazer exatamente o que o nome sugere: Alinhar o endereço inicial da seção em um múltiplo de 16, que é uma exigência da instrução MOVAPS.
{% endhint %}

Um detalhe interessante que vale citar é que apesar da instrução ter sido feita para lidar com um determinado tipo de dado nada impede de nós carregarmos outros dados nos registradores XMM. No exemplo abaixo usei a instrução MOVUPS para mover uma string de 16 bytes com apenas duas instruções:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

void assembly(char *array);

int main(void)
{
  char text[16];
  assembly(text);

  printf("Resultado: %s\n", text);
  return 0;
}
```
{% endtab %}

{% tab title="assembly.asm" %}
```
bits 64
default rel

section .rodata
    string: db "Hello World!", 0, 0, 0, 0

section .text

global assembly
assembly:
    movups xmm0, [string]
    movups [rdi], xmm0
    ret
```
{% endtab %}
{% endtabs %}

### MOVS\(S\|D\) \| Move Scalar \(Single\|Double\)-precision floating-point

```text
MOVSS xmm(n), xmm(n)
MOVSS xmm(n), float(1)
MOVSS float(1), xmm(n)


MOVSD xmm(n), xmm(n)
MOVSD xmm(n), double(1)
MOVSD double(1), xmm(n)
```

Move um único _float/double_ entre registradores XMM, onde o valor estaria contido na _double word_ \(4 bytes\) ou _quadword_ \(8 bytes\) menos significativo do registrador. E também é possível mover de/para memória principal.

### MOVLP\(S\|D\) \| Move Low Packed \(Single\|Double\)-precision floating-point

```text
MOVLPS xmm(n), float(2)
MOVLPS float(2), xmm(n)


MOVLPD xmm(n), double(1)
MOVLPD double(1), xmm(n)
```

A instrução MOVLPS instrução é semelhante à MOVUPS porém carrega/escreve apenas dois floats. No registrador os dois _floats_ ficam armazenados no _quadword_ \(8 bytes\) menos significativo. O _quadword_ mais significativo do registrador não é alterado.

Já MOVLPD faz a mesma operação porém com um _double_ contido no _quadword_ menos significativo.

### MOVHP\(S\|D\) \| Move High Packed \(Single\|Double\)-precision floating-point

```text
MOVHPS xmm(n), float(2)
MOVHPS float(2), xmm(n)


MOVHPD xmm(n), double(1)
MOVHPD double(1), xmm(n)
```

Semelhante a instrução acima porém armazena/ler o valor do registrador XMM no _quadword_ mais significativo. O _quadword_ menos significativo do registrador não é alterado.

### MOVLHPS \| Move Packed Single-precision floating-point Low to High

```text
MOVLHPS xmm(n), xmm(n)
```

Move o _quadword_ \(8 bytes\) menos significativo do registrador fonte \(a direita\) para o _quadword_ mais significativo do registrador destino. O _quadword_ menos significativo do registrador destino não é alterado.

### MOVHLPS \| Move Packed Single-precision floating-point High to Low

```text
MOVHLPS xmm(n), xmm(n)
```

Move o _quadword_ \(8 bytes\) mais significativo do registrador fonte \(a direita\) para o _quadword_ menos significativo do registrador destino. O _quadword_ mais significativo do registrador destino não é alterado.

### MOVMSKP\(S\|D\) \| Move Packed \(Single\|Double\)-precision floating-point mask

```text
MOVMSKPS reg32/64, xmm(n)


MOVMSKPD reg32/64, xmm(n)
```

MOVMSKPS move os bits mais significativos \(MSB\) de cada um dos quatro valores _float_ contido no registrador XMM para os 4 bits menos significativo do registrador de propósito geral. Os outros bits do registrador são zerados.

Já MOVMSKPD faz a mesma coisa porém com os 2 valores _doubles_ contidos no registrador, assim definindo os 2 bits menos significativos do registrador de propósito geral.

Essa instrução pode ser usada com o intuito de verificar o sinal de cada um dos valores _float/double_, tendo em vista que o bit mais significativo é usado para indicar o sinal do número \(**0** caso positivo e **1** caso negativo\).

