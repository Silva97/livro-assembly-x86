---
description: Instruções de operação aritmética do SSE.
---

# Instruções aritméticas

### ADDP(S|D) | Add Packed (Single|Double)-precision floating-point values

```
ADDPS xmm(n), xmm(n)
ADDPS xmm(n), float(4)


ADDPD xmm(n), xmm(n)
ADDPD xmm(n), double(2)
```

Soma 4 números _float_ (ou 2 números _double_) de uma única vez no registrador destino com os quatro números _float_ (ou 2 números _double_) do registrador/memória fonte. Exemplo:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

void assembly(float *array);

int main(void)
{
  float array[4] = {5.0f, 5.0f, 5.0f, 5.0f};
  assembly(array);

  printf("Resultado: %f, %f, %f, %f\n", array[0], array[1], array[2], array[3]);
  return 0;
}
```
{% endtab %}

{% tab title="assembly.asm" %}
```
bits 64
default rel

section .rodata align=16
    sum_array: dd 1.5
               dd 2.5
               dd 3.5
               dd 4.5

section .text

global assembly
assembly:
    movaps xmm0, [rdi]
    addps xmm0, [sum_array]
    movaps [rdi], xmm0
    ret
```
{% endtab %}
{% endtabs %}

### SUBP(S|D) | Subtract Packed (Single|Double)-precision floating-point values

```
SUBPS xmm(n), xmm(n)
SUBPS xmm(n), float(4)


SUBPD xmm(n), xmm(n)
SUBPD xmm(n), double(2)
```

Funciona da mesma forma que a instrução anterior porém faz uma operação de subtração nos valores.

### ADDS(S|D) | Add Scalar (Single|Double)-precision floating-point value

```
ADDSS xmm(n), xmm(n)
ADDSS xmm(n), float(1)


ADDSD xmm(n), xmm(n)
ADDSD xmm(n), double(1)
```

ADDSS faz a adição do _float_ contido no _double word_ (4 bytes) menos significativo do registrador XMM. Já ADDSD faz a adição do _double_ contido na _quadword_ (8 bytes) menos significativa do registrador.

Conforme exemplo abaixo:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

float sum(float x, float y);

int main(void)
{
  printf("Resultado: %f\n", sum(5.0f, 1.5f));
  return 0;
}
```
{% endtab %}

{% tab title="assembly.asm" %}
```
bits 64

section .text

global sum
sum:
    addss xmm0, xmm1
    ret
```
{% endtab %}
{% endtabs %}

### SUBS(S|D) | Subtract Scalar (Single|Double)-precision floating-point value

```
SUBSS xmm(n), xmm(n)
SUBSS xmm(n), float(1)


SUBSD xmm(n), xmm(n)
SUBSD xmm(n), double(1)
```

Funciona da mesma forma que a instrução anterior porém subtraindo os valores.

### MULP(S|D)  | Multiply Packed (Single|Double)-precision floating-point values

```
MULPS xmm(n), xmm(n)
MULPS xmm(n), float(4)


MULPD xmm(n), xmm(n)
MULPD xmm(n), double(2)
```

Funciona como ADDPS/ADDPD porém multiplicando os números ao invés de somá-los.

### MULS(S|D) | Multiply Scalar (Single|Double)-precision floating-point value

```
MULSS xmm(n), xmm(n)
MULSS xmm(n), float(1)


MULSD xmm(n), xmm(n)
MULSD xmm(n), double(1)
```

Funciona como ADDSS/ADDSD porém multiplicando os números ao invés de somá-los.

### DIVP(S|D) | Divide Packed (Single|Double)-precision floating-point values

```
DIVPS xmm(n), xmm(n)
DIVPS xmm(n), float(4)


DIVPD xmm(n), xmm(n)
DIVPD xmm(n), double(2)
```

Funciona como ADDPS/ADDPD porém dividindo os números ao invés de somá-los.

### DIVS(S|D) | Divide Scalar (Single|Double)-precision floating-point value

```
DIVSS xmm(n), xmm(n)
DIVSS xmm(n), float(1)


DIVSD xmm(n), xmm(n)
DIVSD xmm(n), double(1)
```

Funciona como ADDSS/ADDSD porém dividindo os números ao invés de somá-los.

### RCPPS | Compute Reciprocals of Packed Single-precision floating-point values

```
RCPPS xmm(n), xmm(n)
RCPPS xmm(n), float(4)
```

Calcula o valor aproximado do [inverso multiplicativo](https://pt.wikipedia.org/wiki/Inverso_multiplicativo) dos _floats_ no operando fonte (a direita) e armazena os valores no operando destino.

### RCPSS | Compute Reciprocal of Scalar Single-precision floating-point value

```
RCPSS xmm(n), xmm(n)
RCPSS xmm(n), float(1)
```

Calcula o valor aproximado do inverso multiplicativo do _float_ no operando fonte (a direita) e armazena o resultado na _double word_ (4 bytes) menos significativa do operando destino.

### SQRTP(S|D) | Compute square roots of Packed (Single|Double)-precision floating-point values

```
SQRTPS xmm(n), xmm(n)
SQRTPS xmm(n), float(4)


SQRTPD xmm(n), xmm(n)
SQRTPD xmm(n), double(2)
```

Calcula as raízes quadradas dos números _floats/doubles_ no operando fonte e armazena os resultados no operando destino.

### SQRTS(S|D) | Compute square root of Scalar (Single|Double)-precision floating-point value

```
SQRTSS xmm(n), xmm(n)
SQRTSS xmm(n), float(1)


SQRTSD xmm(n), xmm(n)
SQRTSD xmm(n), double(1)
```

Calcula a raiz quadrada do número escalar no operando fonte e armazena o resultado no _float/double _menos significativo do operando destino. Exemplo:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

double my_sqrt(double x);

int main(void)
{
  printf("Resultado: %f\n", my_sqrt(81.0));
  return 0;
}
```
{% endtab %}

{% tab title="assembly.asm" %}
```
bits 64

section .text

global my_sqrt
my_sqrt:
    sqrtsd xmm0, xmm0
    ret
```
{% endtab %}
{% endtabs %}

### RSQRTPS | Compute Reciprocals of square roots of Packed Single-precision floating-point values

```
RSQRTPS xmm(n), xmm(n)
RSQRTPS xmm(n), float(4)
```

Calcula o [inverso multiplicativo](https://pt.wikipedia.org/wiki/Inverso_multiplicativo) das raízes quadradas dos _floats_ no operando fonte, armazenando os resultados no operando destino. Essa instrução é equivalente ao uso de SQRTPS seguido de RCPPS.

### RSQRTSS | Compute Reciprocal of square root of Scalar Single-precision floating-point value

```
RSQRTSS xmm(n), xmm(n)
RSQRTSS xmm(n), float(1)
```

Calcula o inverso multiplicativo da raiz quadrada do número escalar no operando fonte e armazena o resultado no _double word_ menos significativo do operando destino.

### MAXP(S|D) | return maximum of Packed (Single|Double)-precision floating-point values

```
MAXPS xmm(n), xmm(n)
MAXPS xmm(n), float(4)


MAXPD xmm(n), xmm(n)
MAXPD xmm(n), double(2)
```

Compara cada um dos valores contidos nos dois operandos e retorna o maior valor entre os dois.

### MAXS(S|D) | return maximum of Scalar (Single|Double)-precision floating-point value

```
MAXSS xmm(n), xmm(n)
MAXSS xmm(n), float(1)


MAXSD xmm(n), xmm(n)
MAXSD xmm(n), double(1)
```

Compara os dois valores escalares e armazena o maior deles no _float/double _menos significativo do operando destino.

### MINP(S|D) | return minimum of Packed (Single|Double)-precision floating-point values

```
MINPS xmm(n), xmm(n)
MINPS xmm(n), float(4)


MINPD xmm(n), xmm(n)
MINPD xmm(n), double(2)
```

Funciona da mesma forma que MAXPS/MAXPD porém retornando o menor valor entre cada comparação.

### MINS(S|D) | return minimum of Scalar (Single|Double)-precision floating-point value

```
MINSS xmm(n), xmm(n)
MINSS xmm(n), float(1)


MINSD xmm(n), xmm(n)
MINSD xmm(n), double(1)
```

Funciona da mesma forma que MAXSS/MAXSD porém retornando o menor valor entre os dois.
