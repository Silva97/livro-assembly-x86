# Instruções lógicas e de comparação

## Instruções lógicas SSE

### ANDP(S|D) | bitwise logical AND of Packed (Single|Double)-precision floating-point values

```
ANDPS xmm(n), xmm(n)
ANDPS xmm(n), float(4)


ANDPD xmm(n), xmm(n)
ANDPD xmm(n), double(2)
```

Faz uma operação E bit a bit (_bitwise AND_) em cada um dos valores _float/double_ contidos no operando fonte e armazena o resultado no operando destino.

### ANDNP(S|D) | bitwise logical AND NOT of Packed (Single|Double)-precision floating-point values

```
ANDNPS xmm(n), xmm(n)
ANDNPS xmm(n), float(4)


ANDNPD xmm(n), xmm(n)
ANDNPD xmm(n), double(2)
```

Faz uma operação NAND bit a bit em cada um dos valores _float/double_ contidos no operando fonte e armazena o resultado no operando destino.

### ORP(S|D) | bitwise logical OR of Packed (Single|Double)-precision floating-point values

```
ORPS xmm(n), xmm(n)
ORPS xmm(n), float(4)


ORPD xmm(n), xmm(n)
ORPD xmm(n), double(2)
```

Faz uma operação OU bit a bit (_bitwise OR_) em cada um dos valores _float/double_ contidos no operando fonte e armazena o resultado no operando destino.

### XORP(S|D) | bitwise logical XOR of Packed (Single|Double)-precision floating-point values

```
XORPS xmm(n), xmm(n)
XORPS xmm(n), float(4)


XORPD xmm(n), xmm(n)
XORPD xmm(n), double(2)
```

Faz uma operação OU Exclusivo bit a bit (_bitwise eXclusive OR_) em cada um dos valores _float/double_ contidos no operando fonte e armazena o resultado no operando destino.

## Instruções de comparação SSE

As instruções de comparação do SSE recebem um terceiro operando imediato de 8 bits que serve como identificador para indicar qual comparação deve ser efetuada com os valores, onde os valores válidos são de 0 até 7. Na tabela abaixo é indicado cada valor e qual operação de comparação ele representa:

| Valor | Mnemônico | Descrição                                                                                     |
| ----- | --------- | --------------------------------------------------------------------------------------------- |
| 0     | EQ        | Verifica se os valores são iguais.                                                            |
| 1     | LT        | Verifica se o primeiro operando é menor que o segundo.                                        |
| 2     | LE        | Verifica se o primeiro operando é menor ou igual ao segundo.                                  |
| 3     | UNORD     | Verifica se os valores **não** estão ordenados.                                               |
| 4     | NEQ       | Verifica se os valores **não** são iguais.                                                    |
| 5     | NLT       | Verifica se o primeiro operando **não** é menor que o segundo (ou seja, se é igual ou maior). |
| 6     | NLE       | Verifica se o primeiro operando **não** é menor ou igual que o segundo (ou seja, se é maior). |
| 7     | ORD       | Verifica se os valores estão ordenados.                                                       |

Felizmente para facilitar nossa vida os assemblers, incluindo o NASM, adicionam pseudo-instruções que removem o operando imediato e, ao invés disso, usa os mnemônicos apresentados na tabela como _conditional code_ para a instrução. Como é demonstrado no exemplo abaixo:

```nasm
; As duas instruções abaixo são equivalentes.

CMPPS xmm1, xmm2, 0
CMPEQPS xmm1, xmm2
```

### CMPP(S|D)/CMPccP(S|D) | Compare Packed (Single|Double)-precision floating-point values

```
CMPPS xmm(n), xmm(n), imm8
CMPPS xmm(n), float(4), imm8


CMPPD xmm(n), xmm(n), imm8
CMPPD xmm(n), double(2), imm8
```

Essa instrução compara cada um dos valores _float/double_ contido nos dois operandos e armazena o resultado da comparação no operando fonte (o primeiro). O valor imediato passado como terceiro operando é um código numérico para identificar qual operação de comparação deve ser executada em cada um dos valores.

O resultado é armazenado como todos os bits ligados (1) caso a comparação seja verdadeira, se não todos os bits estarão desligados (0) indicando que a comparação foi falsa. Cada número _float/double_ tem um resultado distinto no registrador destino.

### CMPS(S|D)/CMPccS(S|D) | Compare Scalar (Single|Double)-precision floating-point value

```
CMPSS xmm(n), xmm(n), imm8
CMPSS xmm(n), float(4), imm8


CMPSD xmm(n), xmm(n), imm8
CMPSD xmm(n), double(2), imm8
```

Funciona da mesma forma que a instrução anterior porém comparando um único valor escalar. O resultado é armazenado no _float/double_ menos significativo do operando fonte.

### COMIS(S|D)/UCOMIS(S|D) | (Unordered) Compare Scalar (Single|Double)-precision floating-point value and set EFLAGS

```
COMISS xmm(n), xmm(n)
COMISS xmm(n), float(1)

UCOMISS xmm(n), xmm(n)
UCOMISS xmm(n), float(1)


COMISD xmm(n), xmm(n)
COMISD xmm(n), double(1)

UCOMISD xmm(n), xmm(n)
UCOMISD xmm(n), double(1)
```

As quatro instruções comparam os dois operandos **escalares** e definem as _status flags_ em EFLAGS de acordo com a comparação sem modificar os operandos. Comportamento semelhante ao da instrução CMP.

Quando uma operação aritmética com números _floats_ resulta em NaN existem dois tipos diferentes:

* _quiet_ NaN **** (QNaN): O valor é apenas definido para NaN sem qualquer indicação de problema.
* _signaling_ NaN (SNaN): O valor é definido para NaN e uma exceção _floating-point invalid-operation_ (`#I`) é disparada caso você execute alguma operação com o valor.

A diferença entre COMISS/COMISD e UCOMISS/UCOMISD é que COMISS/COMISD irá disparar a exceção `#I` se o primeiro operando for QNaN ou SNaN. Já UCOMISS/UCOMISD apenas dispara a exceção se o primeiro operando for SNaN.
