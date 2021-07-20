---
description: Entendendo como a pilha (hardware stack) funciona na arquitetura x86
---

# Pilha

Uma pilha, em inglês _stack_, é uma estrutura de dados LIFO -- _Last In First Out_ -- onde o último dado a entrar é o primeiro a sair. Imagine uma pilha de livros onde você vai colocando um livro sobre o outro e, após empilhar tudo, você resolve retirar um de cada vez. Ao retirar os livros você vai retirando desde o topo até a base, ou seja, os livros saem na ordem inversa em que foram colocados. O que significa que o último livro que você colocou na pilha vai ser o primeiro a ser retirado, isso é LIFO.

### Hardware Stack

Processadores da arquitetura x86 tem uma implementação nativa de uma pilha, que é representada na memória RAM, onde essa pode ser manipulada por instruções específicas da arquitetura ou diretamente como qualquer outra região da memória. Essa pilha normalmente é chamada de _hardware stack_.

O registrador SP/ESP/RSP, _Stack Pointer_, serve como ponteiro para o topo da pilha podendo ser usado como referência inicial para manipulação de valores na mesma. Onde o "topo" nada mais é que o último valor empilhado. Ou seja, o _Stack Pointer_ está sempre apontando para o último valor na pilha.

A manipulação básica da pilha é empilhar \(_push_\) e desempilhar \(_pop_\) valores na mesma. Veja o exemplo na nossa PoC:

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

Na linha `6` empilhamos o valor de RAX na pilha, alteramos o valor na linha `8` mas logo em seguida desempilhamos o valor e jogamos de volta em RAX. O resultado disso é o valor `12345` sendo retornado pela função.

A instrução `pop` recebe como operando um registrador ou endereçamento de memória onde ele deve armazenar o valor desempilhado.

A instrução `push` recebe como operando o valor a ser empilhado. O tamanho de cada valor na pilha também acompanha o barramento interno \(64 bits em 64-bit, 32 bits em _protected mode_ e 16 bits em _real mode_\). Pode-se passar como operando um valor na memória, registrador ou valor imediato.

A pilha "cresce" para baixo. O que significa que toda vez que um valor é inserido nela o valor de ESP é subtraído pelo tamanho em bytes do valor. E na mesma lógica um `pop` incrementa o valor de ESP. Logo as instruções seriam equivalentes aos dois pseudocódigos abaixo \(considerando um código de 32-bit\):

{% tabs %}
{% tab title="push-pseudo.c" %}
```c
ESP = ESP - 4
[ESP] = operando
```
{% endtab %}

{% tab title="pop-pseudo.c" %}
```
operando = [ESP]
ESP = ESP + 4
```
{% endtab %}
{% endtabs %}

