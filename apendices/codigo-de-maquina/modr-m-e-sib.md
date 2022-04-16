---
description: Entendendo os byte ModR/M e SIB.
---

# ModR/M e SIB

Como já foi mencionado anteriormente o byte ModR/M é usado em algumas instruções para especificar o operando na memória ou registrador.

Em Assembly existem dois "tipos" de instruções que recebem dois operandos:

1. As que tem um operando registrador e imediato. Exemplo: `mov eax, 123`&#x20;
2. As que tem um operando na memória ou dois operandos registradores. Exemplos: `mov [ebx], 123` e `mov eax, ebx`.

O primeiro tipo não precisa do byte ModR/M, pois o registrador destino é especificado nos 3 últimos bits do byte do [opcode](opcode.md). Por exemplo o opcode `B8` da instrução `mov eax, 123` é o seguinte em binário: `10111000` Onde o número zero (`000`) é o código para identificar o registrador EAX.

A tabela abaixo mostra o código de cada um dos registradores de propósito geral:

```
| Registrador | Código |
|-------------|--------|
| AL/AX/EAX   | 0      |
| CL/CX/ECX   | 1      |
| DL/DX/EDX   | 2      |
| BL/BX/EBX   | 3      |
| AH/SP/ESP   | 4      |
| CH/BP/EBP   | 5      |
| DH/SI/ESI   | 6      |
| BH/DI/EDI   | 7      |
```

Repare como AL, AX e EAX por exemplo contém o mesmo código. Qual registrador será usado depende do tamanho do operando.

Um jeito mais simples de especificar esse campo no opcode sem precisar lidar com binário é simplesmente somar o opcode "base" (correspondente ao uso de AL/AX/EAX) mais o código do registrador. Por exemplo se a instrução `B8` (`B8 + 0`) corresponde a `mov eax, 123`, então o opcode `BB` (`B8 + 3`) é `mov ebx, 123`. E se eu quiser fazer `mov bx, 123` basta adicionar o prefixo `66` à instrução.

Já as instruções do segundo tipo usam o byte ModR/M para definir o operando destino na memória (no caso de instruções sem o operando registrador) ou os dois operandos. Onde o byte ModR/M consiste nos três campos:

* `MOD` - Os primeiros 2 bits que definem o "modo" do operando R/M.
* `REG` - Os 3 próximos bits que definem o código do operando registrador.
* `R/M` - Os 3 últimos bits que definem o código do operando R/M.

O byte define 2 operandos:

1. Um operando que é sempre um registrador, definido no campo `REG`.
2. Um operando que pode ser um registrador ou operando na memória.

Para que o campo `R/M` defina também o código de um registrador, assim como o `REG`, o valor 3 (`11` em binário) deve ser usado no campo `MOD`.

{% hint style="info" %}
Um adendo sobre o byte ModR/M é que em algumas instruções o campo `REG` é usado como uma extensão do opcode.\
\
É o caso por exemplo das instruções `inc dword [ebx]` (`FF 03`) e `dec dword [ebx]` (`FF 0B`) que contém o mesmo byte de opcode mas fazem operações diferentes.\
\
Repare como o campo R/M é necessário para especificar o operando na memória mas o REG fica "sobrando", por isso os engenheiros da Intel tomaram essa decisão minimamente confusa (~~vulgo gambiarra~~), afim de aproveitar dessa peculiaridade em instruções que precisam de um operando na memória mas não precisam de um operando registrador.
{% endhint %}

Para os demais valores do campo `MOD` os seguintes endereçamentos são feitos de acordo com o valor de `R/M` (em modo de 32-bit):

#### MOD 00

| R/M | Endereçamento       |
| --- | ------------------- |
| 000 | \[eax]              |
| 001 | \[ecx]              |
| 010 | \[edx]              |
| 011 | \[ebx]              |
| 100 | SIB                 |
| 101 | displacement 32-bit |
| 110 | \[esi]              |
| 111 | \[edi]              |

#### MOD 01

| R/M | Endereçamento               |
| --- | --------------------------- |
| 000 | \[eax] + displacement 8-bit |
| 001 | \[ecx] + displacement 8-bit |
| 010 | \[edx] + displacement 8-bit |
| 011 | \[ebx] + displacement 8-bit |
| 100 | SIB + displacement 8-bit    |
| 101 | \[ebp] + displacement 8-bit |
| 110 | \[esi] + displacement 8-bit |
| 111 | \[edi] + displacement 8-bit |

#### MOD 10

| R/M | Endereçamento                |
| --- | ---------------------------- |
| 000 | \[eax] + displacement 32-bit |
| 001 | \[ecx] + displacement 32-bit |
| 010 | \[edx] + displacement 32-bit |
| 011 | \[ebx] + displacement 32-bit |
| 100 | SIB + displacement 32-bit    |
| 101 | \[ebp] + displacement 32-bit |
| 110 | \[esi] + displacement 32-bit |
| 111 | \[edi] + displacement 32-bit |

### Byte SIB

Os endereçamentos com R/M `100` são os que usam o byte SIB (exceto `MOD 11`), que como já foi explicado anteriormente contém os campos **Scale**, **Index** e **Base** que são calculados de maneira equivalente a expressão:

```
base + index * scale
```

Onde o campo **scale** são os 2 primeiros bits, onde seu valor numérico é equivalente aos seguintes fatores de escala:

* `00` - Não multiplica o **index**
* `01` - Multiplica o **index** por 2
* `10` - Multiplica o **index** por 4
* `11` - Multiplica o **index** por 8

Já os campos **index** e **base** contém 3 bits cada e os mesmos armazenam o código dos registradores que serão usados. Os bits dos campos no byte seguem a ordem que o próprio nome sugere. Como em: `SSIIIBBB`.
