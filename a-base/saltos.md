---
description: Desviando o fluxo de execução do código
---

# Saltos

Provavelmente você já sabe o que é um desvio de fluxo de código em uma linguagem de alto nível. Algo como uma instrução `if` que condicionalmente executa um determinado bloco de código, ou um `for` que executa várias vezes o mesmo bloco de código. Tudo isso é possível devido ao desvio do fluxo de código. Vamos a um pseudo-exemplo de um `if`:

```
1. Compare o valor de X com Y
2. Se o valor de X for maior, pule para 4.
3. Adicione 2 ao valor de X
4.
```

Repare que se a comparação no passo 1 der que o valor de X é maior, a instrução no passo 2 faz um desvio para o passo 4. Desse jeito o passo 3 nunca será executado. Porém caso a condição no passo 2 for falsa, isto é, o valor de X não é maior do que o valor de Y então o desvio não irá acontecer e o passo 3 será executado.

Ou seja o passo 3 só será executado sob uma determinada condição. Isso é um código condicional, isso é um `if`. Repare que o resultado da comparação no passo 1 precisa ficar armazenado em algum lugar, e este "lugar" é o registrador FLAGS.

### Salto não condicional

Antes de vermos um desvio de fluxo condicional vamos entender como é o próprio desvio de fluxo em si.\
Na verdade existem muito mais registradores do que os que eu já citei. E um deles é o registrador IP, sigla para _Instruction Pointer_ (ponteiro de instrução). Esse registrador também acompanha o tamanho do barramento interno, assim como os registradores gerais:

| IP      | EIP     | RIP     |
| ------- | ------- | ------- |
| 16 bits | 32 bits | 64 bits |

Assim como o nome sugere o _Instruction Pointer_ serve como um ponteiro para a próxima instrução a ser executada pelo processador. Desse jeito é possível mudar o fluxo do código simplesmente alterando o valor de IP, porém não é possível fazer isso diretamente com uma instrução como a `mov`.

Na arquitetura x86 existem as instruções de _jump_, salto em inglês, que alteram o valor de IP permitindo assim que o fluxo seja alterado. A instrução de _jump_ não condicional, intuitivamente, se chama JMP. Esse desvio de fluxo é algo muito semelhante com a instrução `goto` da linguagem C, inclusive em boa parte das vezes o compilador converte o `goto` para meramente um JMP.

O uso da instrução JMP é feito da seguinte forma:

```nasm
jmp endereço
```

Onde o operando você pode passar um rótulo que o assembler irá converter para o endereço corretamente. Veja o exemplo na nossa PoC:

{% tabs %}
{% tab title="assembly.asm" %}
```nasm
bits 64

global assembly
assembly:
  mov eax, 555
  jmp .end

  mov eax, 333

.end:
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

A instrução na linha 8 nunca será executada devido ao JMP na linha 6.

{% hint style="info" %}
Repare que na linha 10 estamos usando um rótulo local que foi explicado no tópico sobre a [sintaxe do nasm](sintaxe.md).
{% endhint %}

### Registrador FLAGS

O registrador FLAGS também é estendido junto ao tamanho do barramento interno. Então temos:

| FLAGS   | EFLAGS  | RFLAGS  |
| ------- | ------- | ------- |
| 16 bits | 32 bits | 64 bits |

Esse registrador, diferente dos registradores gerais, não pode ser acessado diretamente por uma instrução. O valor de cada **bit** do registrador é testado por determinadas instruções e são ligados e desligados por outras instruções. É testando o valor dos **bits** do registrador FLAGS que as instruções condicionais funcionam.

### Salto condicional

Os _jumps_ condicionais, normalmente referidos como Jcc, são instruções que condicionalmente fazem o desvio de fluxo do código. Elas verificam os valores dos bits do registrador FLAGS e, com base nos valores, será decidido se o salto será tomado ou não. Assim como no caso do JMP as instruções Jcc também recebem como operando o endereço para onde devem tomar o salto **caso** a condição seja atendida. Se ela não for atendida então o fluxo de código continuará normalmente.

Eis a lista dos saltos condicionais mais comuns:

| Instrução | Nome estendido                       | Condição                   |
| --------- | ------------------------------------ | -------------------------- |
| JE        | **J**ump if **E**qual                | Pula se for igual          |
| JNE       | **J**ump if **N**ot **E**qual        | Pula se **não** for igual  |
| JL        | **J**ump if **L**ess than            | Pula se for menor que      |
| JG        | **J**ump if **G**reater than         | Pula se for maior que      |
| JLE       | **J**ump if **L**ess or **E**qual    | Pula se for menor ou igual |
| JGE       | **J**ump if **G**reater or **E**qual | Pula se for maior ou igual |

{% hint style="info" %}
O nome Jcc para se referir aos saltos condicionais vem do prefixo 'J' seguido de 'cc' para indicar uma condição, que é o formato da nomenclatura das instruções.

Exemplo: JLE -- 'J' prefixo, 'LE' condição (_Less or Equal_)

Essa mesma nomenclatura também é usada para as outras instruções condicionais, como por exemplo CMOVcc.
{% endhint %}

A maneira mais comum usada para setar as _flags_ para um salto condicional é a instrução CMP. Ela recebe dois operandos e compara o valor dos dois, com base no resultado da comparação ela seta as _flags_ corretamente. Agora um exemplo na nossa PoC:

{% tabs %}
{% tab title="assembly.asm" %}
```nasm
bits 64

global assembly
assembly:
  mov eax, 0

  mov rbx, 7
  mov rcx, 5  
  cmp rbx, rcx
  jle .end

  mov eax, 1
.end:
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

Na linha 10 temos um _Jump if Less or Equal_ para o rótulo local `.end`, e logo na linha anterior uma comparação entre RBX e RCX. Se o valor de RBX for menor ou igual a RCX, então o salto será tomado e a instrução na linha 12 não será executada. Desta forma temos algo muito parecido com o `if` no pseudocódigo abaixo:

```c
eax = 0;
rbx = 7;
rcx = 5;
if(rbx > rcx){
  eax = 1;
}
return;
```

Repare que a condição para o código ser executado é exatamente o oposto da condição para o salto ser tomado. Afinal de contas a lógica é que caso o salto seja tomado o código **não** será executado.

{% hint style="info" %}
Experimente modificar os valores de RBX e RCX, e também teste usando outros Jcc.
{% endhint %}
