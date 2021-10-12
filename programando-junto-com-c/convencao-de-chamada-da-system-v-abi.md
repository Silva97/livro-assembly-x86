---
description: Aprendendo sobre a convenção de chamada do C usada no Linux.
---

# Convenção de chamada da System V ABI

Sistemas [UNIX-Like](https://pt.wikipedia.org/wiki/Sistema_operacional_tipo_Unix), incluindo o Linux, seguem a padronização da System V ABI (ou SysV ABI). Onde ABI é sigla para _**A**pplication **B**inary** I**nterface _(Interface binária de aplicação) que é basicamente uma padronização que dita como código binário deve ser escrito e executado no sistema operacional. Uma das coisas que a SysV ABI padroniza é a convenção de chamada utilizada em cada arquitetura de processador.

Neste tópico vamos aprender sobre a convenção de chamada da SysV ABI e o tamanho dos tipos de dados usados na linguagem C.

## Convenção de chamada em x86-64

### Registradores

* Os registradores RBP, RBX, RSP e R12 até R15 são considerados como pertencentes a função chamadora. Isto é, se a função que foi chamada precisar modificar esses registradores ela obrigatoriamente precisa preservar seus valores e antes de retornar restaurá-los para o valor anterior. Todos os outros registradores podem ser modificados livremente pela função chamada. Portanto não espere que esses outros registradores tenham seu valor preservado ao chamar uma função.
* A _Direction Flag_ (DF) no [RFLAGS](../aprofundando-em-assembly/flags-do-processador.md) precisa obrigatoriamente estar zerada ao chamar ou retornar de uma função.

### Stack frame

Cada função chamada pode (se precisar) reservar um pedaço [da pilha](../a-base/pilha.md) para ser usada como memória local da função e pode, por exemplo, ser usada para alocar variáveis locais. Esse espaço é chamado de _stack frame_ e o código que aloca e desaloca o _stack frame_ é chamado de prólogo e epílogo respectivamente. Exemplo:

{% code title="assembly.s" %}
```
    .text
    .globl assembly
assembly:
    sub $8, %rsp

    movl $12344, (%rsp)   # var_0
    movl $1, 4(%rsp)      # var_4

    mov (%rsp), %eax
    add 4(%rsp), %eax

    add $8, %rsp
    ret
```
{% endcode %}

O espaço de 128 bytes antes do endereço apontado por RSP é uma região chamada de _**redzone**_ que por convenção pode ser usada por funções folha (_leaf_), que são funções que não chamam outras funções. Ou então pode ser usada em qualquer função onde o valor não precise ser preservado após chamar outra função. 

O endereço entre `-128(%rsp)` e `-1(%rsp)` pode ser usado livremente sem a necessidade de alocar um _stack frame_.

{% hint style="info" %}
Vale lembrar que [CALL](../aprofundando-em-assembly/call-e-ret.md) empilha o endereço de retorno, portanto ao chamar uma função `0(%rsp)` aponta para o endereço de retorno da mesma.
{% endhint %}

### Passagem de parâmetros

Os parâmetros inteiros (e ponteiros) são passados em [registradores de propósito geral](../a-base/registradores-gerais.md) na seguinte ordem: RDI, RSI, RDX, RCX, R8 e R9. Parâmetros _float_ ou _double_ são passados nos registradores XMM0 até XMM7 como [valores escalares](../aprofundando-em-assembly/entendendo-sse/#registradores-xmm) (na parte menos significativa do registrador).

Caso a função precise de mais argumentos e os registradores acabem, os demais argumentos serão empilhados na ordem **inversa**. Por exemplo caso uma função precise de 9 argumentos inteiros eles seriam definidos na seguinte ordem pela função chamadora:

```
mov $1, %rdi
mov $2, %rsi
mov $3, %rdx
mov $4, %rcx
mov $5, %r8
mov $6, %r9

push $9
push $8
push $7
call my_function
add $24, %esp
```

Assim que a função fosse chamada `8(%rsp)`, `16(%rsp)` e `24(%rsp)` apontariam para os argumentos 7, 8 e 9 respectivamente.

{% hint style="warning" %}
A função **chamadora** (_caller_) precisa garantir que o último valor empilhado esteja em um endereço alinhado por 16 bytes.

A função **chamadora** é a responsável por remover os argumentos empilhados da pilha.
{% endhint %}

### Retorno de valores

* No caso do retorno de estruturas (_structs_) a função chamadora precisa alocar o espaço necessário para a _struct_ e passar o endereço do espaço no registrador RDI como se fosse o primeiro argumento para a função (os outros argumentos usam RSI em diante). A função então precisa retornar o mesmo endereço passado por RDI em RAX.
* O retorno de valores inteiros e ponteiros é feito no registrador RAX.
* Valores _float_ ou _double_ são retornados no registrador XMM0 na parte menos significativa.

## Convenção de chamada em IA-32

### Registradores

* Os registradores EBX, EBP, ESI, EDI e ESP precisam ter seus valores preservados pela função chamada. Os demais registradores de propósito geral podem ser usados livremente.
* A _Direction Flag_ (DF) no EFLAGS precisa obrigatoriamente estar zerada ao chamar ou retornar de uma função.

### Stack frame

O _stack frame_ em IA-32 funciona da mesma maneira que o _stack frame_ em x86-64, com a diferença de que não existe _redzone_ em IA-32 e toda função que precisar de memória local precisa obrigatoriamente construir um _stack frame_.

Vale lembrar que cada valor inserido na _stack_ em IA-32 tem 4 bytes de tamanho, enquanto em x86-64 cada valor tem 8 bytes de tamanho.

### Passagem de parâmetros

Os argumentos da função são empilhados na ordem inversa, assim como ocorre em x86-64 quando os registradores acabam. Conforme exemplo:

```
push $4
push $3
push $2
push $1
call my_function
add $16, %esp
```

Assim que a função é chamada `4(%esp)`, `8(%esp)`, `12(%esp)` e `16(%esp)` apontam para os argumentos 1, 2, 3 e 4 respectivamente.

{% hint style="warning" %}
A função **chamadora** precisa garantir que o último valor empilhado esteja em um endereço alinhado por 16 bytes.

A função **chamadora** é a responsável por remover os argumentos empilhados da pilha.
{% endhint %}

### Retorno de valores

* Retorno de _struct_ é feito de maneira semelhante do x86-64. Um ponteiro para a região de memória para gravar os dados da _struct_ é passado como primeiro argumento para a função (o último valor a ser empilhado). É obrigação da função chamada fazer o _pop_ desse ponteiro e retorná-lo em EAX.
* Valores inteiros e ponteiros são retornados em EAX.
* Valores _float_ ou _double_ são retornados em ST0 (ver [Usando instruções da FPU](../aprofundando-em-assembly/usando-instrucoes-da-fpu.md#registradores)).

## Prólogo e epílogo

Existe uma convenção de escrita do prólogo e do epílogo da função que se trata de preservar o antigo valor de ESP/RSP no registrador EBP/RBP, e depois subtrair ESP/RSP para alocar o _stack frame_. Conforme exemplo:

```
example:
    push %rbp
    mov %rsp, %rbp
    sub $16, %rsp
    
    # etc...
    
    mov %rbp, %rsp
    pop %rbp
    ret
```

Também existe a instrução `leave` que pode ser usada no epílogo. Ela basicamente faz a operação de `mov %rbp, %rsp` e `pop %rbp` em uma única instrução (também pode ser usada em 32 e 16 bits atuando com EBP/ESP e BP/SP respectivamente).

Mas como já foi demonstrado em um exemplo mais acima isso não é obrigatório e podemos apenas incrementar e subtrair ESP/RSP no prólogo e no epílogo. Código otimizado gerado pelo GCC costuma apenas fazer isso, já código com a otimização desligada costuma gerar o prólogo e epílogo "clássico". 

## Tamanho dos tipos da linguagem C

A tabela abaixo lista os principais tipos da linguagem C e seu tamanho em bytes no IA-32 e x86-64. Como também exibe em qual registrador o tipo deve ser retornado.

| Tipo                                                                                                       | <p>Tamanho<br>IA-32</p> | <p>Tamanho<br>x86-64</p> | <p>Registrador de retorno<br>IA-32</p> | <p>Registrador de retorno<br>x86-64</p> |
| ---------------------------------------------------------------------------------------------------------- | :---------------------: | :----------------------: | :------------------------------------: | :-------------------------------------: |
| <p>_Bool</p><p>char</p><p>signed char</p><p>unsigned char</p>                                              |            1            |             1            |                   AL                   |                    AL                   |
| <p>short</p><p>signed short</p><p>unsigned short</p>                                                       |            2            |             2            |                   AX                   |                    AX                   |
| <p>int</p><p>signed int</p><p>unsigned int</p><p>long</p><p>signed long</p><p>unsigned long</p><p>enum</p> |            4            |             4            |                   EAX                  |                   EAX                   |
| <p>long long</p><p>signed long long</p><p>unsigned long long</p>                                           |            8            |             8            |                \*EDX:EAX               |                   RAX                   |
| Ponteiros                                                                                                  |            4            |             8            |                   EAX                  |                   RAX                   |
| float                                                                                                      |            4            |             4            |                   ST0                  |                   XMM0                  |
| double                                                                                                     |            8            |             8            |                   ST0                  |                   XMM0                  |
| \*\*long double                                                                                            |            12           |            16            |                   ST0                  |                   ST0                   |

\*No registrador EDX é armazenado os 32 bits mais significativos e em EAX os 32 bits menos significativos.

\*\*O tipo `long double` ocupa na memória o espaço de 12 e 16 bytes por motivos de alinhamento, mas na verdade se trata de um _float_ de 80 bits (10 bytes).
