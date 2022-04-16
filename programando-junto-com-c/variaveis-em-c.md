---
description: Entendendo como variáveis em C são representadas em Assembly.
---

# Variáveis em C

Como já vimos em [A base](https://mentebinaria.gitbook.io/assembly/), variáveis nada mais são do que um pedaço de memória que pode ser manipulado pelo programa. Em C existem diversas nuances em como variáveis são alocadas e mantidas pelo compilador e aqui vamos entender essas diferenças.

{% hint style="info" %}
Na linguagem C existem palavra-chaves que são chamadas de _storage-class specifiers_, onde elas determinam o _storage-class_ de uma variável. Na prática isso determina como a variável deve ser armazenada no arquivo objeto. No C11 existem os seguintes _storage-class specifiers_:

* extern
* static
* \_Thread\_local
* auto (esse é o padrão)
* register
{% endhint %}

## Variáveis globais

As variáveis globais em C são alocadas na seção `.data` ou `.bss`, dependendo se ela foi inicializada ou não. Como no exemplo:

```c
int data_var = 1;
int bss_var;
```

Se compilamos com `gcc main.c -S -o main.s -fno-asynchronous-unwind-tables` obtemos a seguinte saída:

{% code title="main.s" %}
```
	.globl	data_var
	.data
	.align 4
	.type	data_var, @object
	.size	data_var, 4
data_var:
	.long	1
	.comm	bss_var,4,4
```
{% endcode %}

A variável `data_var` foi alocada na seção `.data` e teve seu símbolo exportado com a diretiva `.globl data_var`, que é equivalente a diretiva `global` do NASM.

Já a variável `bss_var` foi declarada com a diretiva `.comm symbol, size, aligment` que serve para declarar _commom symbols_ (símbolos comuns). Onde ela recebe como argumento o nome do símbolo seguido de seu tamanho (em bytes) e opcionalmente um valor de alinhamento. Em arquivos objetos ELF o terceiro argumento de alinhamento é um alinhamento em bytes, nesse exemplo a variável será alocada em um endereço alinhado por 4 bytes.

Já em arquivos objetos PE (do Windows) o alinhamento é um valor em potência de dois, logo para alinhar em 4 bytes deveríamos passar 2 como argumento ( $$2² = 4$$ ). Se a gente passasse 4 como argumento então seria um alinhamento de $$2^4$$ que daria um alinhamento de 16 bytes.

Os símbolos declarados com a diretiva `.comm` que não foram inicializados em qualquer arquivo objeto são alocados na seção `.bss`. Logo nesse caso o uso da diretiva seria equivalente ao uso de `res*` do NASM, com a diferença que no NASM precisamos usar explicitamente na seção onde o espaço será alocado.

### Variável static global

As variáveis globais com _storage-class_ `static` funcionam da mesma maneira que as variáveis globais comum, com a diferença que seu símbolo não é exportado para que possa ser acessado em outro arquivo objeto. Como no exemplo:

```c
static int data_var = 1;
static int bss_var;
```

Onde obtemos a saída:

```
	.data
	.align 4
	.type	data_var, @object
	.size	data_var, 4
data_var:
	.long	1
	.local	bss_var
	.comm	bss_var,4,4
```

Repare que dessa vez o símbolo `data_var` não foi exportado com a diretiva `.globl`, enquanto o `bss_var` foi explicitamente declarado como local com a diretiva `.local` (já que a diretiva `.comm` exporta como global por padrão).

### Variável extern

Variáveis `extern` em C são basicamente variáveis que são definidas em outro módulo. O GAS tem uma diretiva `.extern` que é equivalente a diretiva `extern` do NASM, isto é, indica que o símbolo será definido em outro arquivo objeto. Porém qualquer símbolo não declarado já é considerado externo por padrão pelo GAS. Experimente ver o código de saída do exemplo abaixo:

```c
extern int extern_var;

int main(void)
{
  int x = extern_var;
  return 0;
}
```

Você vai reparar que na função `main` o símbolo `extern_var` foi lido porém ele não foi declarado.

## Variáveis locais

Variáveis locais em C são comumente alocadas no _stackframe_ da função, porém em alguns casos o compilador também pode reservar um registrador para armazenar o valor da variável.

Em C existe o _storage-class_ `register` que serve como um "pedido" para o compilador alocar aquela variável de forma que o acesso a mesma seja o mais rápido possível, que geralmente é em um registrador (daí o nome da palavra-chave). Mas isso não garante que a variável será realmente alocada em um registrador. Na prática o único efeito colateral garantido é que você não poderá obter o endereço na memória daquela variável com o operador de endereço (`&`), e muitas vezes o compilador já vai alocar a variável em um registrador mesmo sem o uso da palavra-chave.

### Variável static local

Variáveis `static` local são armazenadas da mesma maneira que as variáveis `static` global, a única coisa que muda é no ponto de vista do código-fonte em C onde a visibilidade da variável é limitada para o escopo onde ela foi declarada. Isso faz com o que o compilador gere um símbolo de nome único para a variável, como no exemplo abaixo:

{% tabs %}
{% tab title="test.c" %}
```c
int test(void)
{
  static int data_var = 5;
  static int bss_var;

  return data_var + bss_var;
}
```
{% endtab %}

{% tab title="test.s" %}
```
	.data
	.align 4
	.type	data_var.1913, @object
	.size	data_var.1913, 4
data_var.1913:
	.long	5
	.local	bss_var.1914
	.comm	bss_var.1914,4,4
```
{% endtab %}
{% endtabs %}

Repare como `data_var.1913` não teve seu símbolo exportado e `bss_var.1914` foi declarado como local.

## Variáveis \_Thread\_local

O _storage-class_ `_Thread_local` foi adicionado no C11. Assim como o nome sugere ele serve para alocar variáveis em uma região de memória que é local para cada [_thread_](https://pt.wikipedia.org/wiki/Thread\_\(computa%C3%A7%C3%A3o\)) do processo. Vamos analisar o exemplo:

{% tabs %}
{% tab title="test.c" %}
```c
_Thread_local int global_thread_data = 5;
_Thread_local int global_thread_bss;

int test(void)
{
  _Thread_local static int local_thread_data = 5;
  _Thread_local static int local_thread_bss;

  return global_thread_data
    + global_thread_bss
    + local_thread_data
    + local_thread_bss;
}
```
{% endtab %}

{% tab title="test.s" %}
```
	.text
	.globl	global_thread_data
	.section	.tdata,"awT",@progbits
	.align 4
	.type	global_thread_data, @object
	.size	global_thread_data, 4
global_thread_data:
	.long	5
	.globl	global_thread_bss
	.section	.tbss,"awT",@nobits
	.align 4
	.type	global_thread_bss, @object
	.size	global_thread_bss, 4
global_thread_bss:
	.zero	4
	.section	.tdata
	.align 4
	.type	local_thread_data.1915, @object
	.size	local_thread_data.1915, 4
local_thread_data.1915:
	.long	5
	.section	.tbss
	.align 4
	.type	local_thread_bss.1916, @object
	.size	local_thread_bss.1916, 4
local_thread_bss.1916:
	.zero	4
	.text
	.globl	test
	.type	test, @function
test:
	endbr64
	pushq	%rbp
	movq	%rsp, %rbp
	movl	%fs:global_thread_data@tpoff, %edx
	movl	%fs:global_thread_bss@tpoff, %eax
	addl	%eax, %edx
	movl	%fs:local_thread_data.1915@tpoff, %eax
	addl	%eax, %edx
	movl	%fs:local_thread_bss.1916@tpoff, %eax
	addl	%edx, %eax
	popq	%rbp
	ret
```
{% endtab %}
{% endtabs %}

No Linux, em x86-64, a região de memória local para cada _thread_ (_thread-local storage_ - TLS) fica no segmento apontado pelo [registrador de segmento](../aprofundando-em-assembly/registradores-de-segmento.md) FS, por isso os valores das variáveis estão sendo lidos desse segmento.

Repare que as seções são diferentes, `.tdata` (equivalente a `.data` só que _thread-local_) e `.tbss` (equivalente a `.bss`) são utilizadas para armazenar as variáveis.

O sufixo `@tpoff` (_thread pointer offset_) usado nos símbolos indica que o _offset_ do símbolo deve ser calculado levando em consideração a TLS como endereço de origem. Normalmente o _offset_ é calculado com o segmento de dados "normal" como origem.

## Lidando com os tipos da linguagem C

Agora já entendemos onde e como as variáveis são alocadas em C, só falta entender "o que" está sendo armazenado.

### Arrays e strings

O tipo _array_ em C é meramente uma sequência de variáveis do mesmo tipo na memória. Por exemplo podemos inicializar um `int arr[4]` na sintaxe do GAS da seguinte forma:

```c
arr: .long 1, 2, 3, 4
```

Onde os valores `1`, `2`, `3` e `4` são despejados em sequência.

Em C não existe um tipo _string_ porém por convenção as _strings_ são uma _array_ de `char` onde o último `char` contém o valor zero (chamado de terminador nulo). Esse último caractere `'\0'` é usado para denotar o final da _string_ e funções da libc que lidam com _strings_ esperam por isso. Exemplos:

```c
string1: .ascii "Hello World", 0
string2: .ascii "Hello World\0"
string3: .asciz "Hello World"
```

As três _strings_ acima são equivalentes na sintaxe do GAS.

Sobre a passagem de _arrays_ (incluindo obviamente _strings_) como argumento para uma função, isso é feito passando um ponteiro para o primeiro elemento da _array_.

### Ponteiros

Ponteiros em C, na arquitetura x86/x86-64, são traduzidos meramente como o _offset_ do objeto na memória. O segmento não é especificado como parte do valor do ponteiro.

Experimente ler o código de saída do seguinte programa:

```c
#include <stdio.h>

_Thread_local int my_var = 111;

int main(void)
{
  int *test = &my_var;
  *test = 777;

  printf("%d, %d\n", my_var, *test);
}
```

A leitura do endereço de `my_var` vai ser compilada para algo como:

```
movq	%fs:0, %rax
addq	$my_var@tpoff, %rax
movq	%rax, -8(%rbp)

# Com otimização ligada o GCC usa LEA:

movq	%fs:0, %rax
leaq	my_var@tpoff(%rax), %rdi
```

Onde primeiro é obtido o endereço do início do segmento FS que depois é somado ao _offset_ de `my_var`. Assim obtendo o endereço efetivo da variável na memória.

### Estruturas

As estruturas em C são compiladas de forma que os valores dos campos da estrutura são dispostos em sequência na memória, seguindo a mesma ordem que foram declarados na estrutura. Existe a possibilidade do GCC adicionar alguns bytes extras no final da estrutura afim de manter o alinhamento dos dados, esses bytes extras são chamados de _padding_. Exemplo:

```c
#include <stdio.h>

typedef struct
{
  int x;
  char y;
} my_test_t;

my_test_t test = {
    .x = 5,
    .y = 'A',
};

int main(void)
{
  printf("%d, %c | sizeof: %zu\n", test.x, test.y, sizeof test);
}
```

Isso produziria o seguinte código para a inicialização da variável `test`:

```c
	.globl	test
	.data
	.align 8
	.type	test, @object
	.size	test, 8
test:
	.long	5
	.byte	65
	.zero	3
```

Repare a diretiva `.zero 3` que foi usada para despejar 3 bytes zero no final da estrutura, afim de alinhar a mesma em 4 bytes. No total a estrutura acaba tendo 8 bytes de tamanho: 4 bytes do `int`, 1 byte do `char` e 3 bytes de _padding_.

### Unions

As _unions_ são bem simples, são alocadas para o tamanho do maior tipo declarado para a _union_. Por exemplo:

```c
typedef union
{
  int x;
  char y;
} my_test_t;
```

Essa _union_ é alocada na memória da mesma forma que um `int`, que tem 4 bytes de tamanho.
