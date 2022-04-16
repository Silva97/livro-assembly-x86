---
description: >-
  Aprendendo sobre as convenções de chamada usadas no Windows (x64, cdecl e
  stdcall).
---

# Convenções de chamada no Windows

O Windows tem suas próprias convenções de chamadas e o objetivo desse tópico é aprender sobre as três principais que dizem respeito à linguagem C.

## Convenção de chamada x64

Essa é a convenção de chamada padrão usada em x86-64 e portanto é essencial aprendê-la caso vá programar no Windows diretamente em Assembly.

### Registradores

Os registradores RBX, RBP, RDI, RSI, RSP, R12 até R15 e XMM6 até XMM15 devem ser preservados pela função **chamada** (_callee_). Caso a função chamada precise alterar o valor de algum desses registradores ela tem a obrigação de preservar o valor anterior e restaurá-lo antes de retornar.

Os demais registradores são considerados voláteis, isto é, podem ter seu valor alterado quando uma chamada de função é efetuada. A função chamada pode modificar o valor dos registradores voláteis livremente.

### Passagem de parâmetros

* Os primeiros quatro argumentos inteiros ou ponteiros são passados nos seguintes registradores e na mesma ordem: RCX, RDX, R8 e R9. Os demais argumentos devem ser empilhados na ordem **inversa**.
* Os primeiros quatro argumentos _float_ ou _double_ são passados nos registradores XMM0 até XMM3 como [valores escalares](../aprofundando-em-assembly/entendendo-sse/#registradores-xmm). Os demais também são empilhados na ordem inversa.
* _Structs_ e _unions_ de 8, 16, 32 ou 64 bits são passados como se fossem inteiros do respectivo tamanho. Se forem de outro tamanho a função chamadora deve então passar um ponteiro para a _struct/union_ que será armazenada em uma memória alocada pela própria função chamadora. Essa memória **deve** estar em um endereço alinhado por 16 bytes.

A função **chamadora** (_caller_) é responsável por alocar um espaço de 32 bytes na pilha chamado de _shadow space_. Ele é alocado com o intuito de ser usado pela função chamada (_callee_) para armazenar os parâmetros passados em registradores caso seja necessário, por exemplo caso a função chamada precise usar esses registradores com outro intuito. Esse espaço vem antes mesmo do primeiro parâmetro empilhado.

Exemplo de protótipo de função:

```c
int sum(int a, int b, int c, int d, int e, int f);
```

Assim que a função fosse chamada ECX, EDX, R8D e R9D armazenariam os parâmetros `a`, `b`, `c` e `d` respectivamente. O parâmetro `f` seria empilhado seguido do parâmetro `e`.

O `0(%rsp)` seria o endereço de retorno. O espaço entre `8(%rsp)` e `40(%rsp)` é o _shadow space_. `40(%rsp)` apontaria para o parâmetro `e`, enquanto `48(%rsp)` apontaria para o parâmetro `f`. Como na demonstração abaixo:

```
mov %ecx, 8(%rsp)  # Armazenando o parâmetro A no shadow space
mov %edx, 16(%rsp) # Parâmetro B
mov %r8d, 24(%rsp) # Parâmetro C
mov %r9d, 32(%rsp) # Parâmetro D

# Parâmetro E: 40(%rsp)
# Parâmetro F: 48(%rsp)
```

### Retorno de valores

* Valores inteiros e ponteiros são retornados em RAX.
* Valores _float_ ou _double_ são retornados no registrador XMM0.
* O retorno de _structs_ é feito com a função chamadora alocando o espaço de memória necessário para a _struct_, ela então passa o ponteiro para esse espaço como primeiro argumento para a função em RCX. A função chamada (_callee_) deve retornar o mesmo ponteiro em RAX.

## Convenção de chamada cdecl

A convenção de chamada `__cdecl` é a convenção padrão usada em código escrito em C na arquitetura IA-32 (x86).

### Registradores

Apenas os registradores EAX, ECX e EDX são considerados voláteis, ou seja, registradores que podem ser modificados livremente pela função chamada. Todos os demais registradores precisam ser preservados e restaurados antes do retorno da função.

### Passagem de parâmetros

Todos os parâmetros são passados na pilha e devem ser empilhados na ordem **inversa**. A função **chamadora** (_caller_) é a responsável por remover os argumentos da pilha após a função retornar.

Exemplo:

```
push $3
push $2
push $1
call my_function
add $12, %esp

# 12 é o tamanho em bytes dos três valores empilhados
```

### Retorno de valores

* Valores inteiros ou ponteiros são retornados em EAX.
* Valores _float_ ou _double_ são retornados em ST0.
* O retorno de _structs_ ocorre da mesma maneira que na convenção de chamada x64. Com a diferença que o primeiro argumento é, obviamente, passado na pilha.

## Convenção de chamada stdcall

A convenção de chamada `__stdcall` é a utilizada para chamar funções da [WinAPI](https://pt.wikipedia.org/wiki/API\_do\_Windows).

### Registradores

Assim como na `__cdecl` os registradores EAX, ECX e EDX são voláteis e os demais devem ser preservados pela função chamada.

### Passagem de parâmetros

Todos os argumentos são passados na pilha na ordem inversa. A função **chamada** (_callee_) é a responsável por remover os argumentos da pilha. Exemplo:

```
push $3
push $2
push $1
call my_function
```

### Retorno de valores

O retorno de valores funciona da mesma maneira que o retorno de valores da `__cdecl`.
