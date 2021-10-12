---
description: Entendendo a execução de código em C no ambiente hosted.
---

# Ambiente hosted

Na especificação da linguagem C é descrito dois ambientes de execução de código: Os ambientes _hosted_ e _freestanding_. Neste tópico vamos entender alguns pontos em relação a como funciona a estrutura e a execução de um programa em C no ambiente _hosted_.

O ambiente _hosted_ essencialmente é o ambiente de execução de um código em C que executa sobre um sistema operacional. Nesse ambiente é esperado que haja suporte para múltiplas _threads_ e todos os recursos descritos na especificação da biblioteca padrão (libc). A inicialização do programa ocorre quando a função **main** é chamada e antes de inicializar o programa é esperado que todos os objetos com [_storage-class_](variaveis-em-c.md) `static` estejam inicializados.

## A função main

A função **main** pode ser escrita com um dos dois protótipos abaixo:

```c
int main(void)
{
  // ...
}
```

```c
int main(int argc, char *argv[])
{
  // ...
}
```

Ou qualquer outro protótipo que seja equivalente a um desses. Como por exemplo `char **argv` também seria válido por ter equivalência a `char *argv[]`. Também pode-se usar qualquer nome de parâmetro, `argc` e `argv` são apenas sugestões.

O primeiro parâmetro passado para a função **main** indica o número de argumentos e o segundo é uma _array_ de ponteiros para `char` onde cada índice na _array_ é um argumento e `argv[argc]` é um ponteiro NULL.

Se o tipo de retorno da função **main** for `int` (ou equivalente), o valor de retorno da primeira chamada para **main** é equivalente a chamar a função **exit** passando esse valor como argumento.

## C startup code

{% hint style="info" %}
Os detalhes de implementação descritos aqui são baseados no código-fonte da glibc e podem ser diferentes em outras implementações da libc. Consulte [as referências](../metadados/referencias.md#glibc) para ver a lista de completa de arquivos fonte consultados.
{% endhint %}

O código na glibc responsável pela inicialização do programa é chamado de _C startup_ (CSU). Ele se encarrega de obter os argumentos de linha de comando, inicializar o TLS, executar o código na seção `.init` dentre outras tarefas de inicialização do programa.

O arquivo `start.S` é o que declara o símbolo `_start`, ou seja, a função de _entry point_ do programa. A última chamada nessa função é para outra função chamada `__libc_start_main` que recebe o endereço da função **main** como primeiro argumento. Depois de algumas inicializações essa função chama a **main**, obtém o valor retornado em EAX e passa como argumento para a função responsável por finalizar o programa no sistema operacional (`exit_group` no Linux e `ExitProcess` no Windows).

Todos esses códigos estão em arquivos objetos pré-compilados no seu sistema operacional. Eles são _linkados_ por padrão quando você invoca o GCC mas não são _linkados_ por padrão se você chamar o _linker_ (`ld`) diretamente.

No meu Linux o arquivo objeto `Scrt1.o` ("crt" é sigla para "_C runtime_") é o que contém o _entry point_ (código do `start.S`). Os arquivos `crti.o` e `crtn.o` contém o prólogo e o epílogo, respectivamente, para as seções `.init` e `.fini`.

No meu Linux esses arquivos estão na pasta `/usr/lib/x86_64-linux-gnu/` e sugiro que consulte o conteúdo dos mesmos com a ferramenta **objdump**, como por exemplo:

```
$ objdump -d /usr/lib/x86_64-linux-gnu/Scrt1.o
```

### Fazendo seu próprio startup code

{% hint style="info" %}
Apenas para fins de curiosidade e dar uma noção mais "palpável" de como isso ocorre, irei ensinar aqui como você pode desabilitar a _linkedição_ do CSU e programar uma versão personalizada do mesmo no Linux. Não recomendo que isso seja feito em um programa de verdade tendo em vista que você perderá diversos recursos que o _C runtime_ padrão da glibc provém.
{% endhint %}

Use o seguinte código de teste:

{% tabs %}
{% tab title="start.s" %}
```
STDOUT_FILENO = 1
SYS_WRITE = 1
SYS_EXIT_GROUP = 231

    .section .rodata
init_msg:
    .string "* Initializing...\n"
    MSG_LENGHT = . - init_msg

    .text
    .globl _start
_start:
    mov $STDOUT_FILENO, %rdi
    lea init_msg(%rip), %rsi
    mov $MSG_LENGHT, %rdx
    mov $SYS_WRITE, %rax
    syscall         # write(STDOUT_FILENO, init_msg, MSG_LENGTH)

    pop %rdi        # argc: RDI
    mov %rsp, %rsi  # argv: RSI
    call main

    mov %rax, %rdi
    mov $SYS_EXIT_GROUP, %rax
    syscall         # exit_group( main(argc, argv) )
```
{% endtab %}

{% tab title="main.c" %}
```c
#include <stdio.h>

int main(int argc, char **argv)
{
  printf("argc = %d\n", argc);

  for (int i = 0; i < argc; i++)
  {
    printf("argv[%d] = '%s'\n", i, argv[i]);
  }

  // Esperamos que argv[argc] seja um ponteiro NULL
  printf("argv[argc] = %s\n", argv[argc]);
  return 0;
}
```
{% endtab %}
{% endtabs %}

Compile com:

```
$ as start.s -o crt1.o
$ gcc main.c -o main.o -c
$ gcc *.o -o test -nostartfiles
```

A opção `-nostartfiles` desabilita a _linkedição_ dos arquivos objeto de inicialização.

O que o nosso `start.s` está fazendo é simplesmente chamar a syscall `write` para escrever uma mensagem na tela, chama a função **main** passando `argc` e `argv` como argumentos e depois chama a syscall `exit_group` passando como argumento o retorno da função **main**.

{% hint style="info" %}
No Linux, logo quando o programa é iniciado no _entry point_, o valor contendo o número de argumentos de linha de comando (argc) está em `(%rsp)`. E logo em seguida (RSP+8) está o início da _array_ de ponteiros para os argumentos de linha de comando, terminando com um ponteiro NULL.
{% endhint %}

Experimente rodar `objdump -d test` nesse executável "customizado" e depois compare compilando com o CSU comum. Verá que o programa comum contém diversas funções que foram _linkadas_ nele.

## Seções .init e .fini

As seções `.init` e `.fini` contém funções construída nos arquivos `crti.o` e `crtn.o`.

O propósito da função em `.init` é chamar todas as funções na _array_ de ponteiros localizada em outra seção chamada `.init_array`. Essas funções são invocadas antes da chamada para a função **main**.

Já a função em `.fini` invoca as funções da _array_ na seção `.fini_array` na finalização do programa (após **main** retornar ou na chamada de `exit()`).

No GCC você pode adicionar funções para serem invocadas na inicialização do programa com o atributo `constructor`, e para a finalização do programa com o atributo `destructor`. Experimente ver o código Assembly do exemplo abaixo:

```c
#include <stdio.h>

__attribute__((constructor))
void constructor1(void)
{
  puts("* constructor 1");
}

__attribute__((constructor))
void constructor2(void)
{
  puts("* constructor 2");
}

__attribute__((destructor))
void destructor1(void)
{
  puts("* destructor 1");
}

__attribute__((destructor))
void destructor2(void)
{
  puts("* destructor 2");
}

int main(void)
{
  puts("* main");
}
```

Ao [ver o Assembly gerado](./#vendo-o-codigo-de-saida-do-gcc) do programa acima irá notar que os endereços das funções são despejados nas seções `.init_array` e `.fini_array`, como em:

```
	.section	.init_array,"aw"
	.align 8
	.quad	constructor1
```

## Funções de saída

### exit

Quando a função `exit()` é invocada (ou **main** retorna), funções registradas pela função `atexit()` são executadas. Onde as funções registradas devem seguir o protótipo:

```c
void funcname(void);
```

As funções registradas por `atexit()` são invocadas na ordem inversa a que foram registradas.

### quick_exit

Quando a função `quick_exit()` é invocada o programa é finalizado sem invocar as funções registradas por `atexit()` e sem executar quaisquer _handlers_ de sinal.

As funções registradas por `at_quick_exit` são invocadas na ordem inversa em que foram registradas.

Exemplo:

```c
#include <stdio.h>
#include <stdlib.h>

void func_atexit(void)
{
  puts("* exiting...");
}

void func_at_quick_exit(void)
{
  puts("* Quick exiting...");
}

int main(void)
{
  atexit(func_atexit);
  at_quick_exit(func_at_quick_exit);

  puts("* main");
  // quick_exit(EXIT_SUCCESS);
  return 0;
}
```

Experimente executar o programa acima e depois recompilar com a chamada para **quick_exit** na linha 20.

{% hint style="info" %}
A quantidade máxima de funções que podem ser registradas com **atexit** ou **at_quick_exit** depende da implementação. Mas a especificação do C11 garante que no mínimo 32 funções podem ser registradas por cada uma destas funções.
{% endhint %}

### \_Exit

A função `_Exit()` finaliza a execução do programa sem executar quaisquer funções registradas por **atexit** ou **at_quick_exit**. Também não executa nenhum _handler_ de sinal.
