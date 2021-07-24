# Instruções com inteiros 128-bit

### PAVGB/PAVGW \| Compute average of packed unsigned \(byte\|word\) of integers

```text
PAVGB xmm(n), xmm(n)
PAVGB xmm(n), ubyte(16)


PAVGW xmm(n), xmm(n)
PAVGW xmm(n), uword(8)
```

Calcula a média da soma de todos os valores dos dois operandos somados. PAVGB calcula a média somando 16 bytes em cada operando, enquanto PAVGW soma 8 _words_ em cada um.

### PEXTRW \| Extract word

```text
PEXTRW reg32/64, xmm(n), imm8
PEXTRW uword(1), xmm(n), imm8  ; Adicionado no SSE4
```

Copia uma das 8 _words_ contidas no registrador XMM e armazena no [registrador de propósito geral](../../a-base/registradores-gerais.md) de 32 ou 64 bits. O valor é movido para os 16 bits menos significativos do registrador e todos os outros bits são zerados. Também é possível armazenar a _word_ diretamente na memória principal.

O operando imediato é um valor entre **0** e **7** que indica o índice da _word_ no registrador XMM. Apenas os 3 bits menos significativos do valor são considerados, os demais são ignorados.

### PINSRW \| Insert word

```text
PINSRW xmm(n), reg32, imm8
PINSRW xmm(n), uword(1), imm8
```

Copia uma _word_ dos 16 bits menos significativos do registrador de propósito geral no segundo operando e armazena em uma das _words_ no registrador XMM. Também é possível ler a _word_ da memória principal.

Assim como no caso do PEXTRW o operando imediato serve para identificar o índice da _word_ no registrador XMM.

### PMAXUB/PMAXUW \| Maximum of packed unsigined \(byte\|word\) of integers

```text
PMAXUB xmm(n), xmm(n)
PMAXUB xmm(n), ubyte(16)


PMAXUW xmm(n), xmm(n)      ; Adicionado no SSE4
PMAXUW xmm(n), uword(8)    ; Adicionado no SSE4
```

Compara os bytes/_words_ não-sinalizados dos dois operandos _packed_ e armazena o maior deles em cada uma das comparações no operando destino \(o primeiro\).

### PMINUB/PMINUW \| Minimum of packed unsigned \(byte\|word\) of integers

```text
PMINUB xmm(n), xmm(n)
PMINUB xmm(n), ubyte(16)


PMINUW xmm(n), xmm(n)      ; Adicionado no SSE4
PMINUW xmm(n), uword(8)    ; Adicionado no SSE4
```

Faz o mesmo que a instrução anterior porém armazenando o menor número de cada comparação.

### PMAXS\(B\|W\|D\) \| Maximum of packed signed \(byte\|word\|doubleword\) integers

```text
PMAXSB xmm(n), xmm(n)       ; Adicionado no SSE4
PMAXSB xmm(n), byte(16)     ; Adicionado no SSE4


PMAXSW xmm(n), xmm(n)
PMAXSW xmm(n), word(8)


PMAXSD xmm(n), xmm(n)       ; Adicionado no SSE4
PMAXSD xmm(n), dword(4)     ; Adicionado no SSE4
```

Faz o mesmo que PMAXUB/PMAXUW porém considerando o valor como sinalizado. Também há a instrução PMAXSD que compara quatro _double words_ \(4 bytes\) empacotados.

### PMINS\(B\|W\) \| Minimum of packed signed \(byte\|word\) integers

```text
PMINSB xmm(n), xmm(n)       ; Adicionado no SSE4
PMINSB xmm(n), byte(16)     ; Adicionado no SSE4


PMINSW xmm(n), xmm(n)
PMINSW xmm(n), word(8)
```

Faz o mesmo que PMAXSB/PMAXSW porém retornando o menor valor de cada comparação.

### PMOVMSKB \| Move byte mask

```text
PMOVMSKB reg32/64, xmm(n)
```

Armazena nos 16 bits menos significativos do registrador de propósito geral cada um dos bits mais significativos \(MSB\) de todos os bytes contidos no registrador XMM.

### PMULHW/PMULHUW \| Multiply packed \(unsigned\) word integers and store high result

```text
PMULHW xmm(n), xmm(n)
PMULHW xmm(n), uword(8)


PMULHUW xmm(n), xmm(n)
PMULHUW xmm(n), uword(8)
```

Multiplica cada uma das _words_ dos operandos onde o resultado temporário da multiplicação é de 32 bits de tamanho. Porém armazena no operando destino somente a _word_ mais significativa do resultado da multiplicação.

PMULHW faz uma multiplicação sinalizada, enquanto PMULHUW faz uma multiplicação não-sinalizada.

### PSADBW \| Compute sum of absolute differences

```text
PSADBW xmm(n), xmm(n)
PSADBW xmm(n), ubyte(16)
```

Calcula a diferença absoluta dos bytes dos dois operandos e armazena a soma de todas as diferenças.

A diferença dos 8 bytes menos significativos é somada e armazenada na _word_ menos significativa do operando destino.  Já a diferença dos 8 bytes mais significativos é somada e armazenada na quinta _word_ \(índice **4**, bits \[79:64\]\) do operando destino. Todas as outras _words_ do registrador XMM são zeradas.

### MOVDQA \| Move aligned double quadword

```text
MOVDQA xmm(n), xmm(n)
MOVDQA xmm(n), qword(2)
MOVDQA qword(2), xmm(n)
```

Move dois _quadwords_ \(8 bytes\) entre registradores XMM ou de/para memória RAM. O endereço na memória precisa estar alinhado a 16 se não uma exceção \#GP será disparada.

### MOVDQU \| Move unaligned double quadword

```text
MOVDQU xmm(n), xmm(n)
MOVDQU xmm(n), qword(2)
MOVDQU qword(2), xmm(n)
```

Faz o mesmo que a instrução anterior porém o alinhamento da memória não é necessário, porém essa instrução é menos performática caso acesse um endereço desalinhado.

### PADD\(B\|W\|D\|Q\) \| Packed \(byte\|word\|doubleword\|quadword\) add

```text
PADDB xmm(n), xmm(n)
PADDB xmm(n), byte(16)


PADDW xmm(n), xmm(n)
PADDW xmm(n), word(8)


PADDD xmm(n), xmm(n)
PADDD xmm(n), dword(4)


PADDQ xmm(n), xmm(n)
PADDQ xmm(n), qword(2)
```

Soma os bytes, _words_, _double words_ ou _quadwords_ dos operandos e armazena no operando destino.

### PSUBQ \| Packed quadword subtract

```text
PSUBQ xmm(n), xmm(n)
PSUBQ xmm(n), qword(2)
```

Faz o mesmo que a instrução PADDQ porém com uma operação de subtração.

### PMULUDQ \| Multiply packed unsigned doubleword integers

```text
PMULUDQ xmm(n), xmm(n)
PMULUDQ xmm(n), dword(4)
```

Multiplica o primeiro \(índice 0\) e o terceiro \(índice 2\) _doublewords_ dos operandos e armazena o resultado como _quadwords_ no operando destino. O resultado da multiplicação entre os primeiros _doublewords_ é armazenado no _quadword_ menos signfiicativo do operando destino, enquanto a multiplicação entre os terceiros _doublewords_ é armazenada no _quadword_ mais significativo.

Exemplo:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>
#include <inttypes.h>

void mularray(uint64_t *output, uint32_t *array);

int main(void)
{
  uint32_t array[] = {3, 1, 2, 1};
  uint64_t output[2];
  mularray(output, array);

  printf("Resultado: %" PRIu64 ", %" PRIu64 "\n", output[0], output[1]);
  return 0;
}
```
{% endtab %}

{% tab title="assembly.asm" %}
```
bits 64
default rel

section .rodata align=16
    mul_values: dd 2, 3, 4, 5

section .text

global mularray
mularray:
    movdqa xmm0, [mul_values]
    pmuludq xmm0, [rsi]
    movdqa [rdi], xmm0
    ret
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
RDI é o primeiro ponteiro recebido como argumento e RSI o segundo.
{% endhint %}

### PSLLDQ \| Shift double quadword left logical

```text
PSLLDQ xmm(n), imm8
```

Faz uma operação de [_logical shift_](https://en.wikipedia.org/wiki/Logical_shift) _left_ com os dois _quadwords_ do registrador XMM. O número de vezes que o _shift_ deve ser feito é especificado pelo operando imediato de 8 bits. Os bits menos significativos são zerados.

### PSRLDQ \| Shift double quadword right logical

```text
PSRLDQ xmm(n), imm8
```

Faz o mesmo que a instrução anterior porém com um _shift right_. Os bits mais significativos são zerados.

