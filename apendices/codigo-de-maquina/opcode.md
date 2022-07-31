---
description: Entendendo o opcode da instrução.
---

# Opcode

Como já foi dito antes existem opcodes cujo os 3 últimos bits são usados para identificar o registrador usado na instrução. Opcodes nesse estilo de codificação são usados para instruções que só precisam usar um registrador. Por exemplo `mov eax, 123` cujo o opcode é `B8`.

Já em instruções que usam o [byte ModR/M](modr-m-e-sib.md) os dois bits menos significativos do opcode tem um significado especial, que são chamados de bit <mark style="color:green;">**D**</mark> (_direction bit_) e <mark style="color:red;">**S**</mark> (_size bit_). Conforme ilustração:

![Representação dos bits de um opcode.](../../.gitbook/assets/opcode-ilustração.png)

### **BIT D**

A função do bit <mark style="color:green;">**D**</mark> é indicar a direção para onde a operação está sendo executada. Se do REG para o R/M ou vice-versa. Repare nas instruções abaixo e seus respectivos opcodes:

```nasm
mov eax, [ebx] ; 8B03
mov [ebx], eax ; 8903
```

Convertendo os opcodes `8B` e `89` para binário dá para notar um fato interessante:

```
8B -> 10001011
89 -> 10001001
```

A única diferença entre os opcodes é que em um o bit <mark style="color:green;">**D**</mark> está ligado e no outro não. Quando o bit <mark style="color:green;">**D**</mark> está ligado o campo REG é usado como operando destino e o campo R/M usado como fonte. E quando ele está desligado é o inverso: o campo R/M é o destino e o REG é o fonte. Obviamente o mesmo também se aplica se o R/M também for um registrador.

Por exemplo a instrução `xor eax, eax` pode ser escrita em código de máquina como `31 C0` ou `33 C0`. Como no campo REG e no campo R/M são os mesmos registradores não faz diferença qual é o fonte e qual é o destino, a operação executada será a mesma. Usando um _disassembler_ como o **ndisasm** dá para notar isso:

```
felipe@silva-lenovo:~$ echo -ne "\x31\xC0\x33\xC0" > tst
felipe@silva-lenovo:~$ ndisasm -b32 tst
00000000  31C0              xor eax,eax
00000002  33C0              xor eax,eax
```

### **BIT S**

O bit <mark style="color:red;">**S**</mark> é usado para definir o tamanho do operando, onde:

* `0` -> Indica que o operando é de 8-bit
* `1` -> Indica que o operando é do tamanho do **operand-size**.

Repare por exemplo a instrução `30 C0`:

```
felipe@silva-lenovo:~$ echo -ne "\x30\xC0" > tst
felipe@silva-lenovo:~$ ndisasm -b32 tst
00000000  30C0              xor al,al
```

Onde `31 C0` (com o bit <mark style="color:red;">**S**</mark> ligado) usa o operando de 32-bit EAX. Mas `30 C0` usa o operando de 8-bit AL.

Repare também no seguinte caso:

```
felipe@silva-lenovo:~$ echo -ne "\x66\x30\xC0\x66\x31\xC0" > tst
felipe@silva-lenovo:~$ ndisasm -b32 tst
00000000  6630C0            o16 xor al,al
00000003  6631C0            xor ax,ax
```

Veja que ao usar o prefixo `66` ([_operand-size override_](atributos-e-prefixos.md#atributo-operand-size)) em `31 C0` o registrador AX é utilizado. Mas esse prefixo é ignorado em instruções cujo o bit <mark style="color:red;">**S**</mark> esteja desligado. Por isso o **ndisasm** faz o _disassembly_ da instrução ainda como `xor al, al`. Embora ele adicione um `o16` ali para denotar o uso (inútil) do prefixo.
