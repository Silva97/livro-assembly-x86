---
description: Entendendo as funções em C do ponto de vista do Assembly.
---

# Funções em C

A linguagem C tem algumas variações à respeito de funções e o objetivo deste tópico é explicar, do ponto de vista do baixo-nível, como elas funcionam.

## Entendendo os protótipos

As funções na linguagem C têm protótipos que servem como uma "assinatura" indicando quais parâmetros a função recebe e qual tipo de valor ela retorna. Um exemplo:

```c
int add(int x, int y);
```

Esse protótipo já nos dá todas as informações necessárias que saibamos como fazer a chamada da função e como obter seu valor de retorno, desde que nós conheçamos a [convenção de chamada](convencao-de-chamada-da-system-v-abi.md) utilizada. Os parâmetros são considerados da esquerda para a direita, logo o parâmetro `x` é o primeiro e o parâmetro `y` é o segundo. Na convenção de chamada da SysV ABI esses argumentos estariam em EDI e ESI, respectivamente. E o retorno seria feito em EAX.

Existem alguns protótipos um pouco diferentes que vale explicar aqui para deixar claro seu entendimento. Como este:

```c
void do_something(int a);
```

De acordo com a especificação do C11 uma expressão do tipo `void` é um tipo cujo o valor não existe e **deve** ser ignorado. Funções assim são compiladas retornando sem se preocupar em modificar o valor de RAX \(ou qualquer outro registrador que poderia ser usado para retornar um valor\) e portanto não se deve esperar que o valor nesse registrador tenha alguma informação útil.

```c
int do_something(void);
```

Quando `void` é usado no lugar da lista de parâmetros ele tem o significado especial de indicar que aquela função não recebe parâmetro algum ao ser chamada.

```c
int do_something();
```

Embora possa ser facilmente confundido com o caso acima, onde se usa `void` na lista de parâmetros, na verdade esse protótipo de função não diz que a função não recebe parâmetros. Na verdade esse é um protótipo que não especifica quais tipos ou quantos parâmetros a função recebe, logo o compilador aceita que a função seja chamada passando qualquer tipo e qualquer quantidade de parâmetros, inclusive sem parâmetro algum também. Veja o exemplo:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

int do_something();

int main(void)
{
  printf("Resultado: %d\n", do_something(1, 2, 3, 4.5f, "teste"));
}
```
{% endtab %}

{% tab title="main.s" %}
```
	movq	.LC1(%rip), %rax
	leaq	.LC0(%rip), %rcx
	movq	%rax, %xmm0
	movl	$3, %edx
	movl	$2, %esi
	movl	$1, %edi
	movl	$1, %eax
	call	do_something@PLT
```
{% endtab %}
{% endtabs %}

Na convenção de chamada da SysV ABI os argumentos para esse tipo de função são passados da mesma maneira que uma chamada com o protótipo "normal". A única diferença é que a função recebe um argumento extra no registrador AL indicando quantos registradores de vetor foram utilizados para passar argumentos de ponto-flutuante. Nesse exemplo apenas um argumento era um _float_ e por isso há a instrução `movl $1, %eax` indicando esse número. Experimente usar mais argumentos _float_ ou não passar nenhum para ver se o número passado em AL como argumento irá mudar de acordo.

```c
int do_something(int x, ...);
```

Funções com [argumentos variáveis](https://en.wikipedia.org/wiki/Variadic_function) também seguem a mesma regra de chamada do que foi mencionado acima.

## Funções static

Funções _static_ são visíveis apenas no mesmo módulo em que elas foram declaradas, ou seja, seu símbolo não é exportado. Exemplo:

```c
static int add(int a, int b)
{
  return a + b;
}
```

## Function specifiers

Existem dois especificadores de função no C11, onde eles são:

### inline

O especificador `inline` é uma sugestão para que a chamada para a função seja a mais rápida possível. Isso tem o efeito colateral no GCC de inibir a geração de código para a função em Assembly. Ao invés disso as instruções da função são geradas no local onde ela foi chamada, e portanto o símbolo da função nunca é de fato declarado.

{% hint style="warning" %}
O GCC, mesmo para uma função _inline_, ainda vai gerar o código para a chamada da função caso as otimizações estejam desligadas e isso vai acabar produzindo um erro de referência por parte do _linker_. Lembre-se de sempre ligar as otimizações de código quando estiver usando funções _inline_.
{% endhint %}

### \_Noreturn

Funções com o especificador `_Noreturn` nunca devem retornar para a função chamadora. Quando esse especificador é utilizado o compilador irá gerar código assumindo que a função nunca retorna. Como podemos ver no exemplo abaixo compilado com `-O2`:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>
#include <stdlib.h>

_Noreturn void goodbye(const char *msg)
{
  puts(msg);
  exit(EXIT_SUCCESS);
}

int main(void)
{
  goodbye("Sayonara onii-chan! ^-^");
}
```
{% endtab %}

{% tab title="main.s" %}
```
	.text
	.p2align 4
	.globl	goodbye
	.type	goodbye, @function
goodbye:
	endbr64
	pushq	%rax
	popq	%rax
	subq	$8, %rsp
	call	puts@PLT
	xorl	%edi, %edi
	call	exit@PLT
	.size	goodbye, .-goodbye
	.section	.rodata.str1.1,"aMS",@progbits,1
.LC0:
	.string	"Sayonara onii-chan! ^-^"
	.section	.text.startup,"ax",@progbits
	.p2align 4
	.globl	main
	.type	main, @function
main:
	endbr64
	pushq	%rax
	popq	%rax
	leaq	.LC0(%rip), %rdi
	subq	$8, %rsp
	call	goodbye
```
{% endtab %}
{% endtabs %}

## Funções aninhadas

_Nested functions_ é uma extensão do GCC que permite declarar funções aninhadas. O símbolo de uma função aninhada é gerado de maneira semelhante ao símbolo de uma variável local com _storage-class_ `static`_._ Exemplo:

```c
int calc(int a, int b)
{
  int add(int a, int b)
  {
    return a + b;
  }

  return add(a, b) + add(3, 4);
}
```

## Atributos de função

Os atributos de função é uma extensão do GCC que permite modificar algumas propriedades relacionadas à uma função. Se define atributos para uma função usando a palavra-chave `__attribute__` e entre dois parênteses uma lista de atributos separado por vírgula. Exemplo:

```c
__attribute__((cdecl))
int add(int a, int b)
{
  return a + b;
}
```

Alguns atributos recebem parâmetros onde estes devem ser adicionados dentro de mais um par de parênteses, se assemelhando a sintaxe de uma chamada de função. Exemplo: `__attribute__((section (".another"), cdecl))`.

Abaixo alguns atributos que podem ser usados na arquitetura x86 e acho interessante citar aqui:

### ms\_abi, sysv\_abi, cdecl, stdcall, fastcall, thiscall

Esses atributos fazem com que o compilador gere o código da função usando a convenção de chamada [ms\_abi](convencoes-de-chamada-no-windows.md#convencao-de-chamada-x64), [sysv\_abi](convencao-de-chamada-da-system-v-abi.md), [cdecl](convencoes-de-chamada-no-windows.md#convencao-de-chamada-cdecl), [stdcall](convencoes-de-chamada-no-windows.md#convencao-de-chamada-stdcall), fastcall ou thiscall respectivamente. Também é útil usá-los em protótipos de funções onde a função utiliza uma convenção de chamada diferente da padrão.

{% hint style="info" %}
Os atributos `cdecl`, `stdcall`, `fastcall` e `thiscall` são ignorados em 64-bit.
{% endhint %}

### section \("name"\)

Por padrão o GCC irá adicionar o código das funções na seção `.text`, porém é possível usar o atributo `section` para que o compilador adicione o código da função em outra seção. Como no exemplo abaixo:

{% tabs %}
{% tab title="main.c" %}
```c
__attribute__((section(".another")))
int add(int a, int b)
{
  return a + b;
}
```
{% endtab %}

{% tab title="main.s" %}
```
	.section	.another,"ax",@progbits
	.p2align 4
	.globl	add
	.type	add, @function
add:
	endbr64
	leal	(%rdi,%rsi), %eax
	ret
```
{% endtab %}
{% endtabs %}

### naked

O atributo `naked` é usado para desativar a geração do [prólogo e epílogo](convencao-de-chamada-da-system-v-abi.md#prologo-e-epilogo) para a função. Isso é útil para se escrever funções usando Inline Assembly dentro das mesmas.

### target \("option1", "option2", ...\)

Esse atributo serve para personalizar a geração de código do compilador para uma função específica, permitindo selecionar quais instruções serão utilizadas ao gerar o código. Também é possível adicionar o prefixo `no-` para desabilitar alguma tecnologia e impedir que o compilador gere código para ela. Por exemplo `__attribute__((target ("no-sse"))` desativaria o uso de instruções ou registradores [SSE](../aprofundando-em-assembly/entendendo-sse/) na função.

Alguns dos possíveis alvos para arquitetura x86 são:

| Ativar as instruções | Desativar as instruções |
| :--- | :--- |
| 3dnow | no-3dnow |
| 3dnowa | no-3dnowa |
| abm | no-abm |
| adx | no-adx |
| aes | no-aes |
| avx | no-avx |
| avx2 | no-avx2 |
| avx5124fmaps | no-avx5124fmaps |
| avx5124vnniw | no-avx5124vnniw |
| avx512bitalg | no-avx512bitalg |
| avx512bw | no-avx512bw |
| avx512cd | no-avx512cd |
| avx512dq | no-avx512dq |
| avx512er | no-avx512er |
| avx512f | no-avx512f |
| avx512ifma | no-avx512ifma |
| avx512pf | no-avx512pf |
| avx512vbmi | no-avx512vbmi |
| avx512vbmi2 | no-avx512vbmi2 |
| avx512vl | no-avx512vl |
| avx512vnni | no-avx512vnni |
| avx512vpopcntdq | no-avx512vpopcntdq |
| mmx | no-mmx |
| sse | no-sse |
| sse2 | no-sse2 |
| sse3 | no-sse3 |
| sse4 | no-sse4 |
| sse4.1 | no-sse4.1 |
| sse4.2 | no-sse4.2 |
| sse4a | no-sse4a |
| ssse3 | no-ssse3 |

## PLT e GOT

Já vimos alguns exemplos de código chamando funções da libc, essas funções porém estão em uma biblioteca dinâmica e não dentro do executável. A resolução do endereço \(_symbol binding_\) das funções na biblioteca é feito em tempo de execução onde os endereços são salvos na seção GOT \(_Global Offset Table_\).

A seção PLT \(_Procedure Linkage Table_\) simplesmente armazena saltos para os endereços armazenados na GOT. Por isso o GCC gera chamadas para funções da libc assim:

```c
call	puts@PLT
```

O sufixo `@PLT` indica que o endereço do símbolo está na seção PLT. Onde nessa seção há uma instrução `jmp` para o endereço que será resolvido em tempo de execução na GOT. Algo parecido com a ilustração abaixo:

```text
# Esse código não funciona, é apenas uma ilustração.

  .section .got
real_puts_address.got:
  .quad 0
real_printf_address.got:
  .quad 0

  .section .plt
puts.plt:
  jmp *real_puts_address.got
printf.plt:
  jmp *real_printf_address.got

  .data
message:
  .asciz "Hello World!"

  .text
my_func:
  lea message(%rip), %rdi
  call puts.plt
  
  ret
```

Na sintaxe do NASM o equivalente ao uso do sufixo com `@` do GAS é a palavra-chave `wrt` \(_With Reference To_\), conforme exemplo:

```text
extern puts

section .data
    message: db "Hello World!", 0

section .text
global assembly
assembly:
    lea rdi, [rel message]
    call puts wrt ..plt

    ret
```



