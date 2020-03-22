---
description: Acessando a memória RAM
---

# Endereçamento

O acesso a operandos na memória principal é feito definindo alguns fatores que, após serem calculados pelo processador, resultam no endereço físico que será utilizado a partir do barramento de endereço para acessar aquela região da memória.  
Do ponto de vista do programador são apenas algumas somas e multiplicações.

{% hint style="info" %}
O endereçamento de um operando também pode ser chamado de endereço efetivo, ou em inglês, _effective address_.
{% endhint %}

{% hint style="danger" %}
Não tente ler ou modificar a memória com nossa PoC ainda.  
No final do tópico eu falo sobre a instrução LEA que pode ser usada para testar o endereçamento.
{% endhint %}

### Endereçamento em IA-16

No código de máquina da arquitetura IA-16 existe um byte chamado ModR/M que serve para especificar algumas informações relacionadas ao acesso de \(R\)egistradores e/ou \(M\)emória.  
O endereçamento em IA-16 é totalmente especificado neste byte, e ele nos permite fazer um cálculo no seguinte formato:  
`REG + REG + DESLOCAMENTO`

Onde `REG` seria o nome de um registrador e `DESLOCAMENTO` um valor numérico também somado ao endereço.  
Os registradores `BX, BP, SI e DI` podem ser utilizados. Enquanto o deslocamento é um valor de 8 ou 16 bits.

Neste cálculo um dos registradores é usado como base, o endereço inicial, e o outro é usado como índice, um valor numérico a ser somado assim como o deslocamento.  
`BX e BP` são usados para base enquanto `SI e DI` são usados para índice. Perceba que não é possível somar `base+base` e nem `índice+índice`  
Alguns exemplos para facilitar o entendimento:

```text
mov [bx],           ax ; Correto!
mov [bx+si],        ax ; Correto!
mov [bp+di],        ax ; Correto!
mov [bp+si],        ax ; Correto!
mov [bx+di + 0xa1], ax ; Correto!
mov [si],           ax ; Correto!
mov [0x1a],         ax ; Correto!

mov [dx],    ax ; ERRADO!
mov [bx+bp], ax ; ERRADO!
mov [si+di], ax ; ERRADO!
```

### Endereçamento em IA-32

Em IA-32 o código de máquina tem também o byte SIB, que é um novo modo de endereçamento.  
Enquanto em IA-16 nós temos apenas uma base e um índice, em IA-32 nós ganhamos também um fator de escala.

O fator de escala é basicamente um número que irá multiplicar o valor de índice.

* A escala pode ser 1, 2, 4 ou 8.
* O registrador de índice pode ser qualquer um dos registradores gerais **exceto** ESP.
* O registrador de base pode ser qualquer registrador geral.
* O deslocamento pode ser de 8 ou 32 bits.

Exemplos:

```text
mov [edx],                      eax ; Correto!
mov [ebx+ebp],                  eax ; Correto!
mov [esi+edi],                  eax ; Correto!
mov [esp+ecx],                  eax ; Correto!
mov [ebx*4 + 0x1a],             eax ; Correto!
mov [ebx + ebp*8 + 0xab12cd34], eax ; Correto!
mov [esp + ebx*2],              eax ; Correto!
mov [0xffffaaaa],               eax ; Correto!

mov [esp*2], eax   ; ERRADO!
```

{% hint style="info" %}
SIB é sigla para _**S**cale, **I**ndex and **B**ase_. Que são os três valores usados para calcular o endereço efetivo.
{% endhint %}

### Endereçamento em x86-64

Em x86-64 segue a mesma premissa de IA-32, com alguns adendos:

* É possível usar registradores de 32 ou 64 bit.
* Os registradores de R8 a R15 ou R8D a R15D podem ser usados como base ou índice.
* Não é possível mesclar 32 e 64 bits.

Exemplos:

```text
mov [rbx], rax           ; Correto!
mov [ebx], rax           ; Correto!
mov [r15 + r10*4], rax   ; Correto!
mov [r15d + r10d*4], rax ; Correto!

mov [r10 + r15d], rax    ; ERRADO!
mov [rsp*2],      rax    ; ERRADO!
```

### Truque do nasm

Cuidado para não se confundir em relação ao fator de escala. Veja por exemplo esta instrução 64-bit:

```text
mov [rbx*3], rax
```

Apesar de 3 não ser um valor válido de escala o nasm irá montar o código sem apresentar erros. Isto acontece porque ele converteu a instrução para a seguinte:

```text
mov [rbx + rbx*2], rax
```

Ele usa RBX tanto como base como também índice, e usa o fator de escala 2.  
Resultando no mesmo valor que se multiplicasse RBX por 3.  
Este é um truque do nasm que pode levar ao erro, por exemplo:

```text
mov [rsi + rbx*3], rax
```

Desta vez acusaria erro já que a base foi explicitada.  
Lembre-se que os fatores de escala válidos são 1, 2, 4 ou 8.

### Instrução LEA

```text
lea registrador, [endereço]
```

A instrução LEA, sigla para _Load Effective Address_, calcula o endereço efetivo do segundo operando e armazena o resultado do cálculo em um registrador.  
Esta instrução pode ser útil para testar o cálculo do _effective address_ e ver os resultados usando nossa PoC, conforme exemplo abaixo:

{% tabs %}
{% tab title="assembly.asm" %}
```text
bits 64

global assembly
assembly:
  mov rbx, 5
  mov rcx, 10
  lea eax, [rcx + rbx*2 + 5]
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

