---
description: Aprendendo a usar o x87 para fazer cálculos.
---

# Usando instruções da FPU

Podemos usar a FPU para fazer cálculos com valores de ponto flutuante. A arquitetura x86 segue a padronização [IEEE-754](https://pt.wikipedia.org/wiki/IEEE\_754) para a representação de valores de ponto flutuante.

{% hint style="info" %}
Apenas algumas instruções da FPU serão ensinadas aqui, não sendo uma lista completa.

Um adendo que normalmente compiladores de C não trabalham com valores de ponto flutuante desta maneira em x86-64 porque a arquitetura x86 hoje em dia tem maneiras mais eficientes de fazer esses cálculos. Isso será demonstrado no próximo tópico.
{% endhint %}

### Registradores

As instruções da FPU trabalham com os registradores de **st0** até **st7**, são 8 registradores de 80 bits de tamanho cada. Juntos eles formam uma _stack_ (pilha) onde você pode empilhar valores para trabalhar com eles ou desempilhar para armazenar o resultado das operações em algum lugar.

O empilhamento de valores funciona colocando o novo valor em **st0** e todos os outros valores anteriores são "empurrados" para os registradores posteriores. Um exemplo bem leviano dessa operação:

```
st0 = 10
st1 = 20
st2 = 30

* é feito um push do valor 40

st0 = 40
st1 = 10
st2 = 20
st3 = 30

* é feito um pop, o valor 40 é pego.

st0 = 10
st1 = 20
st2 = 30
```

Detalhe que só é possível usar esses registradores em instruções da FPU, algo como esse código está errado:

```nasm
mov eax, st1
```

### Formato das instruções

As instruções da FPU todas começam com um prefixo **F**, e as que operam com valores inteiros (convertendo DE ou PARA inteiro) também tem uma letra **I** após a letra **F**. Por fim, instruções que fazem o _pop_ de um valor da pilha, isto é, remove o valor de lá, terminam com um sufixo **P**. Entendendo isso fica muito mais fácil identificar o que cada mnemônico significa e assim você não perde tempo tentando decorar uma sopa de letrinhas, se essas letras existem é porque tem um significado.

{% hint style="info" %}
Caso tenha vindo de uma arquitetura RISC, geralmente o termo _load_ é usado para a operação em que você carrega um valor da memória para um registrador. Já _store_ é usado para se referir a operação contrária, do registrador para a memória.

Nesse caso as operações podem ser feita entre registradores da FPU também, conforme será explicado.
{% endhint %}

Fazer _load_ de um valor é basicamente carregar um valor da memória para a pilha em **st0**, é como um _push_ quando estamos falando da pilha convencional. A diferença aqui é a maneira como o valor é colocado na pilha, como já foi explicado anteriormente.

Já o _store_ é pegar o valor da pilha, mais especificamente em **st0**, e armazenar em algum lugar da memória. Algumas instruções _store_ permitem armazenar o valor em outro registrador da FPU.

Aqui eu vou ensinar a usar a FPU mas sem diretamente trabalhar com a linguagem C e os tipos **float** ou **double**, pois como já foi mencionado, não é assim que o compilador trabalha com cálculos de ponto flutuante.

Vou usar a notação `memXXfp`e `memXXint` para especificar valores na memória que sejam _float_ ou inteiro, respectivamente. Onde XX seria o tamanho do valor em bits. Já a notação `st(i)` será usada para se referir a qualquer registrador de **st0** até **st7**. O `st(0)`seria o registrador **st0** especificamente.

### FINIT | Initialization

```nasm
finit
```

Normalmente vamos usar essa instrução antes de começar a usar a FPU, pois ela reseta a FPU para o estado inicial. Dessa forma quaisquer operações anteriores com a FPU são descartadas e podemos começar tudo do zero. Assim não é necessário, por exemplo, a gente limpar a pilha da FPU toda vez que terminar as operações com ela. Basta rodar essa instrução antes de usá-la.

### FLD, FILD | (Integer) Load

```
fld mem32fp
fld mem64fp
fld mem80fp
fld st(i)

fild mem16int
fild mem32int
fild mem64int
```

A instrução `fld` carrega um valor _float_ de 32, 64 ou 80 bits para **st0**. Repare como é possível dar _load_ em um dos registradores da pilha, o que torna possível retrabalhar com valores anteriormente carregados. Se você rodar `fld st0` estará basicamente duplicando o último valor carregado.

Já `fild` carrega um valor inteiro sinalizado de 16, 32 ou 64 bits o convertendo para _float_ de 64 bits.

### Load Constant

Existem várias instruções para dar _push_ de valores constantes na pilha da FPU, e elas são:

| Instrução | Valor                           |
| --------- | ------------------------------- |
| FLD1      | +1.0                            |
| FLDZ      | +0.0                            |
| FLDL2T    | log2(10)                        |
| FLDL2E    | log2(e)                         |
| FLDPI     | Valor de PI. (3.1415 blabla...) |
| FLDLG2    | log10(2)                        |
| FLDLN2    | logE(2)                         |

### FST, FSTP | Store (and Pop)

```
fst mem32fp
fst mem64fp
fst st(i)

fstp mem32fp
fstp mem64fp
fstp mem80fp
fstp st(i)
```

Pega o valor _float_ **** de **st0** e copia para o operando destino. A versão com o sufixo **P** também faz o _pop_ do valor da _stack_, sendo possível dar _store_ em um _float_ de 80 bits somente com essa instrução.

### FIST, FISTP | Integer Store (and Pop)

```
fist mem16int
fist mem32int

fistp mem16int
fistp mem32int
fistp mem64int
```

Pega o valor em **st0**, converte para inteiro sinalizado e armazena no operando destino. Só é possível dar _store_ em um inteiro de 64 bits na versão da instrução que faz o _pop_.

Só com essas instruções já podemos converter um _float_ para inteiro e vice-versa. Conforme exemplo:

{% tabs %}
{% tab title="assembly.asm" %}
```nasm
bits 64

section .data
  num: dq 23.87

section .bss
  result: resd 1

section .text
global assembly
assembly:
  finit
  fld   qword [num]
  fistp dword [result]

  mov eax, [result]
  ret
```
{% endtab %}

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
{% endtabs %}

Se você rodar esse teste irá notar que o valor foi convertido para 24 já que houve um arredondamento.

### FADD, FADDP, FIADD | (Integer) Add (and Pop)

```
fadd mem32fp
fadd mem64fp
fadd st(0), st(i)
fadd st(i), st(0)

faddp st(i), st(0)
faddp

fiadd mem16int
fiadd mem32int
```

As versões de `fadd` com operando na memória faz a soma do operando com **st0** e armazena o resultado da soma no próprio **st0**. Já `fiadd` com operando em memória faz a mesma coisa, porém convertendo o valor inteiro para _float_ 64 bits antes.

As instruções com registradores fazem a soma e armazenam o resultado no operando mais a esquerda, o operando destino. Enquanto a `faddp` sem operandos soma **st0** com **st1**, armazena o resultado em **st1** e depois faz o _pop_.

Exemplo de soma simples:

{% tabs %}
{% tab title="assembly.asm" %}
```nasm
bits 64

section .data
  num1: dq 24.3
  num2: dq 0.7

section .bss
  result: resd 1

section .text
global assembly
assembly:
  finit
  fld  qword [num1]
  fadd qword [num2]

  fist dword [result]
  mov eax, [result]
  ret
```
{% endtab %}

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
{% endtabs %}

### FSUB, FSUBP, FISUB | (Integer) Subtract (and Pop)

```
fsub mem32fp
fsub mem64fp
fsub st(0), st(i)
fsub st(i), st(0)

fsubp st(i), st(0)
fsubp

fisub mem16int
fisub mem32int
```

Mesma coisa que as instruções acima, só que fazendo uma operação de subtração.

### FDIV, FDIVP, FIDIV | (integer) Division (and Pop)

```
fdiv mem32fp
fdiv mem64fp
fdiv st(0), st(i)
fdiv st(i), st(0)

fdivp st(i), st(0)
fdivp

fidiv mem16int
fidiv mem32int
```

Mesma coisa que **FADD** etc. porém faz uma operação de divisão.

### FMUL, FMULP, FIMUL | (Integer) Multiply (and Pop)

```
fmul mem32fp
fmul mem64fp
fmul st(0), st(i)
fmul st(i), st(0)

fmulp st(i), st(0)
fmulp

fimul mem16int
fimul mem32int
```

Cansei de repetir, já sabe né? Operação de multiplicação.

### FSUBR, FSUBRP, FISUBR | (Integer) Subtract Reverse (and Pop)

Faz a mesma coisa que a família **FSUB** só que com os operandos ao contrário. Conforme ilustração:

```c
a = a - b // fsub etc.
a = b - a // fsubr etc.
```

Ou seja faz a subtração na ordem inversa dos operandos, porém onde o resultado é armazenado continua sendo o mesmo.

### FDIVR, FDIVRP, FIDIVRP | (Integer) Division Reverse (and Pop)

Mesma lógica que as instruções acima, porém faz a divisão na ordem inversa dos operandos.

### FXCH | Exchange

```
fxch st(i)
fxch
```

Seguindo a mesma lógica da instrução `xchg`, troca o valor de **st0** com **st(i)**. A versão da instrução sem operando especificado faz a troca entre **st0** e **st1**.

### FSQRT | Square root

```
fsqrt
```

Calcula a raíz quadrada de **st0** e armazena o resultado no próprio **st0**.

### FABS | Absolute

```
fabs
```

Calcula o valor absoluto de **st0** e armazena em **st0**. Basicamente zera o bit de sinalização do valor.

### FCHS | Change Sign

```
fchs
```

Inverte o sinal de **st0**, se era negativo passa a ser positivo e vice-versa.

### FCOS | Cosine

```
fcos
```

Calcula o cosseno de **st0** que deve ser um valor radiano, e armazena o resultado nele próprio.

### FSIN | Sine

```
fsin
```

Calcula o seno de **st0**, que deve estar em radianos.

### FSINCOS | Sine and Cosine

```
fsincos
```

Calcula o seno e o cosseno de **st0**. O cosseno é armazenado em **st0** enquanto o seno estará em **st1**.

### FPTAN | Partial Tangent

```
fptan
```

Calcula a tangente de **st0** e armazena o resultado no próprio registrador, logo após faz o _push_ do valor 1.0 na pilha. O valor em **st0** para ser calculado deve estar em radianos.

### FPATAN | Partial Arctangent

```
fpatan
```

Calcula o arco-tangente de **st1** dividido por **st0**, armazena o resultado em **st1** e depois faz o _pop_. O resultado tem o mesmo sinal que o operando que estava em **st1**.

$$
st1 = \arctan( st1 \div st0 )
$$

### F2XM1 | 2^x - 1

```
f2xm1
```

Faz o cálculo de 2 elevado a **st0** menos 1, e armazena o resultado em **st0**.

$$
st0 = 2^{st0} - 1
$$

### FYL2X | y \* log2(x)

```
fyl2x
```

Faz esse cálculo aí com logaritmo de base 2:

$$
st1 = st1 \cdot \log_2(st0)
$$

Após o cálculo é feito um _pop_.

### **FYL2XP1 | y \* log2(x + 1)**

```
fyl2xp1
```

Mesma coisa que a instrução anterior porém somando 1 a **st0**.

$$
st1 = st1 \cdot \log_2(st0 + 1)
$$

### FRNDINT | Round to Integer

```
frndint
```

Arredonda **st0** para a parte inteira mais próxima e armazena o resultado em **st0**.

### FPREM, FPREM1 | Partial Reminder

```
fprem
fprem1
```

As duas instruções armazenam a sobra da divisão entre **st0** e **st1** no registrador **st0**. Com a diferença que `fprem1` segue a padronização IEEE-754.

### FCOMI, FCOMIP, FUCOMI, FUCOMIP | Compare

```
fcomi  st(0), st(i)
fcomip st(0), st(i)

fucomi  st(0), st(i)
fucomip st(0), st(i)
```

Faz a comparação entre **st0** e **st(i)** setando as _status flags_ de acordo. A diferença de `fucomi` e `fucomip` é que essas duas verificam se os valores nos registradores não são NaN, sendo o caso a instrução irá disparar uma _exception_ #IA.

### FCMOVcc | Conditional Move

```
fcmovb  st(0), st(i)
fcmove  st(0), st(i)
fcmovbe st(0), st(i)
fcmovu  st(0), st(i)

fcmovnb  st(0), st(i)
fcmovne  st(0), st(i)
fcmovnbe st(0), st(i)
fcmovnu  st(0), st(i)
```

Faz uma operação _move_ condicional levando em consideração as _status flags_.

### Vendo os resultados

Adiantando que um valor _float_ na [convenção de chamada](../programando-junto-com-c/convencao-de-chamada-da-system-v-abi.md) do C é retornado no registrador **XMM0**. Podemos ver o resultado de nossos testes da seguinte forma usando a instrução MOVSS:

{% tabs %}
{% tab title="assembly.asm" %}
```nasm
bits 64

section .data
  num1: dq 3.0
  num2: dq 3.0

section .bss
  result: resd 1

section .text
global assembly
assembly:
  finit
  fld  qword [num1]
  fmul qword [num2]

  fst dword [result]
  movss xmm0, [result]
  ret
```
{% endtab %}

{% tab title="main.c" %}
```c
#include <stdio.h>

float assembly(void);

int main(void)
{
  printf("Resultado: %f\n", assembly());
  return 0;
}
```
{% endtab %}
{% endtabs %}

A instrução [MOVSS](entendendo-sse/instrucoes-de-movimentacao-de-dados.md#movs-s-or-d-or-move-scalar-single-or-double-precision-floating-point) e os registradores XMM serão explicados no [próximo tópico](entendendo-sse/).
