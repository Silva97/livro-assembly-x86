---
description: Pilhas são muito úteis
---

# Pilha

Uma pilha, em inglês _stack_, é uma estrutura de dados LIFO -- _Last In First Out_ -- o que significa que o último dado a entrar é o primeiro a sair.  
Imagine uma pilha de livros onde você vai colocando um livro sobre o outro e, após empilhar tudo, você resolve retirar um de cada vez.  
Ao retirar os livros você vai retirando desde o topo até a base, ou seja, os livros saem na ordem inversa em que foram colocados. O que significa que o último livro que você colocou na pilha vai ser o primeiro a ser retirado, isto é LIFO.

### Hardware Stack

Processadores da arquitetura x86 tem implementações nativas de uma pilha que é representada na memória RAM, onde esta pode ser manipulada por instruções específicas da arquitetura ou diretamente como qualquer outra região da memória.  
Esta pilha normalmente é chamada de _hardware stack_.

O registrador SP/ESP/RSP, _Stack Pointer_, serve como ponteiro para o topo da pilha podendo ser usado como referência inicial para manipulação de valores na mesma.

A manipulação básica da pilha é empilhar\(_push_\) e desempilhar \(_pop_\) valores na mesma. Veja exemplo na nossa PoC:

{% tabs %}
{% tab title="assembly.asm" %}
```text
bits 64

global assembly
assembly:
  mov  rax, 12345
  push rax

  mov rax, 112233
  pop rax
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

Na linha `6` empilhamos o valor de RAX na pilha, alteramos o valor na linha `8` mas logo em seguida desempilhamos o valor e jogamos devolta em RAX.  
O resultado disso seria o valor `12345` sendo retornado pela função.

No caso a instrução `pop` recebe como operando um registrador ou endereçamento de memória onde ele deve armazenar o valor desempilhado.

A instrução `push` recebe como operando o valor a ser empilhado. O tamanho de cada valor na pilha também acompanha o barramento interno. \(64 bits em 64-bit, 32 bits em _protected mode_ e 16 bits em _real mode_\)  
Pode-se passar como operando um valor na memória, registrador ou valor imediato.

{% hint style="info" %}
A pilha "cresce" para baixo, o que significa que toda vez que um valor é inserido nela o valor de ESP é subtraído pelo tamanho em bytes do valor. E na mesma lógica um `pop` incrementa o valor de ESP. 
{% endhint %}

