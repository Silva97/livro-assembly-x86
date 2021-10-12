---
description: Entendendo funções em Assembly
---

# Procedimentos

O conceito de um procedimento nada mais é que um pedaço de código que em determinado momento é convocado para ser executado e, logo em seguida, o processador volta a executar as instruções em sequência. Isso nada mais é que uma combinação de dois desvios de fluxo de código, um para a execução do procedimento e outro no fim dele para voltar o fluxo de código para a instrução seguinte a convocação do procedimento. Veja o exemplo em pseudocódigo:

```
1. Define A para 3
2. Chama o procedimento setarA
3. Compara A e 5
4. Finaliza o código

setarA:
7. Define A para 5
8. Retorna
```

Seguindo o fluxo de execução do código, a sequência de instruções ficaria assim:

```
1. Define A para 3
2. Chama o procedimento setarA
7. Define A para 5
8. Retorna
3. Compara A e 5
4. Finaliza o código
```

Desse jeito se nota que a comparação do passo 3 vai dar positiva porque o valor de A foi setado para 5 dentro do procedimento `setarA`.

Em Assembly x86 temos duas instruções principais para o uso de procedimentos:

| Instrução | Operando | Ação                                           |
| --------- | -------- | ---------------------------------------------- |
| CALL      | endereço | Chama um procedimento no endereço especificado |
| RET       | ???      | Retorna de um procedimento                     |

A esta altura você já deve ter reparado que nossa função `assembly` na nossa PoC nada mais é que um procedimento chamado por uma instrução CALL, por isso no final dela temos uma instrução RET.

Na prática o que uma instrução CALL faz é **empilhar** o endereço da instrução seguinte na _stack_ e, logo em seguida, faz o desvio de fluxo para o endereço especificado assim como um JMP. E a instrução RET basicamente **desempilha** esse endereço e faz o desvio de fluxo para o mesmo. Um exemplo na nossa PoC:

{% tabs %}
{% tab title="assembly.asm" %}
```nasm
bits 64

global assembly
assembly:
  mov eax, 3
  call setarA

  ret

setarA:
  mov eax, 5
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

Na linha 6 damos um `call` no procedimento `setarA` na linha 10, este por sua vez altera o valor de EAX antes de retornar. Após o retorno do procedimento a instrução RET na linha 8 é executada, e então retornando também do procedimento `assembly`.

### O que são convenções de chamadas?

É seguindo essa lógica que "milagrosamente" o nosso código em C sabe que o valor em EAX é o valor de retorno da nossa função `assembly`. Linguagens de alto nível, como C por exemplo, usam um conjunto de regras para definir como uma função deve ser chamada e como ela retorna um valor. Essas regras são a convenção de chamada, em inglês, _calling convention_.

Na nossa PoC a função `assembly` retorna uma variável do tipo `int` que na arquitetura x86 tem o tamanho de 4 bytes e é retornado no registrador EAX. A maioria dos valores serão retornados em alguma parte mapeada de RAX que coincida com o mesmo tamanho do tipo. Exemplos:

| Tipo      | Tamanho em x86-64 | Registrador |
| --------- | ----------------- | ----------- |
| char      | 1 byte            | AL          |
| short int | 2 bytes           | AX          |
| int       | 4 bytes           | EAX         |
| char \*   | 8 bytes           | RAX         |

Por enquanto não vamos ver a convenção de chamada que a linguagem C usa, só estou adiantando isso para que possamos entender melhor como nossa função `assembly` funciona.

{% hint style="warning" %}
Em um código em C não tente adivinhar o tamanho em bytes de um tipo. Para cada arquitetura diferente que você compilar o código, o tipo pode ter um tamanho diferente. Sempre que precisar do tamanho de um tipo use o operador `sizeof`.
{% endhint %}
