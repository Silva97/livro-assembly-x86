---
description: Entendendo a codificação dos registradores em 16-bit, 32-bit e 64-bit
---

# Codificação dos registradores

Em modo de 16-bit e 32-bit cada registrador é identificado usando um número de 3 bits, permitindo assim identificar uma variação de 8 registradores diferentes. Porém vários registradores compartilham do mesmo código, e qual especificamente será usado varia de acordo com a instrução sendo utilizada e o tamanho do operando.

Por exemplo [instruções da FPU](../../aprofundando-em-assembly/usando-instrucoes-da-fpu.md) irão sempre usar algum registrador ST0\~ST7, então o código em uma instrução da FPU será usado para identificar algum deles.

Como por exemplo a instrução `fld st3` que em código de máquina fica `D9 C3`, onde `C3` é o ModR/M:

```
MOD -> 11
REG -> 000
R/M -> 011
```

Repare que essa instrução usa o campo `REG` como extensão do opcode e o R/M é usado para especificar o operando. Não coincidentemente o código 3 (`0b011`) é usado para identificar o registrador ST3.

Já instruções que usam [registradores de propósito geral](../../a-base/registradores-de-proposito-geral.md), qual especificamente será usado depende do tamanho do operando na instrução (veja [Atributos e prefixos](atributos-e-prefixos.md)).

Por exemplo as seguintes instruções compiladas em modo de 64-bit:

```
mov eax, 0x11223344                    b8 44 33 22 11
mov ax, 0x1122                         66 b8 22 11
mov al, 0x11                           b0 11
```

Se convertermos esses opcodes em binário teremos o seguinte:

```
B8 -> 10111000
B0 -> 10110000
```

Esses dois opcodes usam os 3 últimos bits para identificar o registrador. Veja que o mesmo código `000` acabou sendo usado para identificar EAX, AX e AL.

Isso porque na primeira instrução o atributo **operand-size** padrão de 32-bit foi usado, então o registrador EAX é usado na instrução. Já na segunda o prefixo **operand-size override** (byte `66`) foi usado, assim o **operand-size** era de 16-bit e portanto o registrador AX é usado.

Já a última instrução é exclusivamente usada para operandos de 8-bit, e portanto o registrador AL é usado.

## Tabela de códigos

Como já foi explicado no tópico que fala sobre o [prefixo REX](prefixo-rex.md), esse prefixo estende os campos usados em ModR/M, SIB e o campo `REG`do opcode em 1 bit. Daí assim o código usado para identificar o registrador, em modo de 64-bit, tem 4 bits de tamanho.

A tabela abaixo lista os códigos usados para identificar os registradores. Lembrando que o bit mais significativo indica um dos bits do REX ligado, ou seja, só é utilizado em modo de 64-bit.

| Código | Registrador                        |
| ------ | ---------------------------------- |
| `0000` | `AL/AX/EAX/RAX/ST0/MM0/XMM0`       |
| `0001` | `CL/CX/ECX/RCX/ST1/MM1/XMM1`       |
| `0010` | `DL/DX/EDX/RDX/ST2/MM2/XMM2`       |
| `0011` | `BL/BX/EBX/RBX/ST3/MM3/XMM3`       |
| `0100` | `AH/SP/ESP/RSP/ST4/MM4/XMM4`       |
| `0101` | `CH/BP/EBP/RBP/ST5/MM5/XMM5`       |
| `0110` | `DH/SI/ESI/RSI/ST6/MM6/XMM6`       |
| `0111` | `BH/DI/EDI/RDI/ST7/MM7/XMM7`       |
| `1000` | `R8B/R8W/R8D/R8/ST0/MM0/XMM8`      |
| `1001` | `R9B/R9W/R9D/R9/ST1/MM1/XMM9`      |
| `1010` | `R10B/R10W/R10D/R10/ST2/MM2/XMM10` |
| `1011` | `R11B/R11W/R11D/R11/ST3/MM3/XMM11` |
| `1100` | `R12B/R12W/R12D/R12/ST4/MM4/XMM12` |
| `1101` | `R13B/R13W/R13D/R13/ST5/MM5/XMM13` |
| `1110` | `R14B/R14W/R14D/R14/ST6/MM6/XMM14` |
| `1111` | `R15B/R15W/R15D/R15/ST7/MM7/XMM15` |

