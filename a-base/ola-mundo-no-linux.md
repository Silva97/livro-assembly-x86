---
description: Finalmente o Hello World.
---

# Olá mundo no Linux

Geralmente o "Hello World" é a primeira coisa que vemos quando estamos aprendendo uma linguagem de programação. Nesse caso eu deixei por último pois acredito que seria de extrema importância entender todos os conceitos antes de vê-lo, isso evitaria a intuição de ver um código em Assembly como um "código em C mais difícil de ler". Acredito que essa comparação mental involuntária é muito ruim e prejudicaria o aprendizado. Por isso optei por explicar tudo antes mesmo de apresentar o famoso "Hello World".

### Hello World

Desta vez vamos escrever um código em Assembly sem misturar com C, será um executável do Linux \(formato ELF64\) fazendo chamadas de sistema diretamente. Vamos vê-lo logo:

{% code title="hello.asm" %}
```text
bits 64

section .rodata
  msg:     db  `Hello World!\n`
  MSG_SIZE equ $-msg

section .text

global _start
_start:
  mov rax, 1
  mov rdi, 1
  mov rsi, msg
  mov rdx, MSG_SIZE
  syscall             ; write

  mov rax, 60
  xor rdi, rdi
  syscall             ; exit
```
{% endcode %}

Para compilar esse código basta usar o NASM especificando o format **elf64** e desta vez iremos usar o _linker_ do pacote GCC diretamente. O nome do executável é **ld** e o uso básico é bem simples, basta especificar o nome do arquivo de saída com **-o**. Ficando assim:

```text
$ nasm hello.asm -felf64
$ ld hello.o -o hello
$ ./hello
```

Na linha 5 definimos uma constante usando o símbolo **$** para pegar o endereço da instrução atual e subtraímos pelo endereço do rótulo `msg`. Isso resulta no tamanho do texto porque `msg` aponta para o início da string e, como está logo em seguida, **$** seria o endereço do final da string.  
`final - início = tamanho`

### write

| Nome | RAX | RDI | RSI | RDX |
| :--- | :--- | :--- | :--- | :--- |
| write | 1 | file\_descriptor | endereço | tamanho \(em bytes\) |

Como deve ter reparado usamos mais uma _syscall_, que foi a _syscall_ `write`. Essa _syscall_ basicamente escreve dados em um arquivo. O primeiro argumento é um número que serve para identificar o arquivo para o qual queremos escrever os dados.

No Linux a saída e entrada de um programa nada mais é que dados sendo escritos e lidos em arquivos. E isso é feito por três arquivos que estão por padrão abertos em um programa e tem sempre o mesmo _file descriptor_, são eles:

| Nome | File descriptor | Descrição |
| :--- | :--- | :--- |
| stdin | 0 | Entrada de dados \(o que é digitado pelo usuário\) |
| stdout | 1 | Saída padrão \(o que é impresso no terminal\) |
| stderr | 2 | Saída de erro \(também impresso no terminal, porém destinado a mensagens de erro\) |

{% hint style="info" %}
Se quiser ver o código de implementação desta _syscall_ no Linux, pode [ver aqui](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/fs/read_write.c).
{% endhint %}

### Entry point

Reparou que nosso programa tem um símbolo `_start` e que magicamente esse é o código que o sistema operacional está executando primeiro? Isso acontece porque o _linker_ definiu o endereço daquele símbolo como o _entry point_ \(ponto de entrada\) do nosso programa.

O _entry point_ nada mais é o que o próprio nome sugere, o endereço inicial de execução do programa. Eu sei o que você está pensando:

> Então a função main de um programa em C é o _entry point_?

A resposta é não! Um programa em C usando a libc tem uma série de códigos que são executados antes da main. E o primeiro deles, pasme, é uma função chamada `_start` definida pela própria libc.

Na verdade qualquer símbolo pode ser definido como o _entry point_ para o executável, não faz diferença qual nome você dá para ele. Só que `_start` é o símbolo padrão que o **ld** define como _entry point_.

Se você quiser usar um símbolo diferente é só especificar com a opção **-e**. Por exemplo, podemos reescrever nosso Hello World assim:

```text
bits 64

section .rodata
  msg:     db  `Hello World!\n`
  MSG_SIZE equ $-msg

section .text

global _eu_que_mando_no_meu_exec
_eu_que_mando_no_meu_exec:
  mov rax, 1
  mov rdi, 1
  mov rsi, msg
  mov rdx, MSG_SIZE
  syscall

  mov rax, 60
  xor rdi, rdi
  syscall
```

E compilar assim:

```text
$ nasm hello.asm -felf64
$ ld hello.o -o hello -e _eu_que_mando_no_meu_exec
$ ./hello
```

Fácil fazer um "Hello World", né? Ei, o que acha de fazer uns macros para melhorar o uso dessas _syscalls_ aí? Seria interessante também salvar os macros em um arquivo separado e incluir o arquivo com a diretiva `%include`.

