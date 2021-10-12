---
description: Convertendo valores entre float, double e inteiro.
---

# Instruções de conversão

Essas instruções servem para conversão de tipos entre _float_, _double_ e inteiro.

## Conversão entre double e float

### CVTPS2PD | Convert packed single-precision floating-point values to packed double-precision floating-point values

```
CVTPS2PD xmm(n), xmm(n)
CVTPS2PD xmm(n), float(2)
```

Converte dois valores _float_ do operando fonte (segundo) em dois valores _double_ no operando destino (primeiro).

### CVTPD2PS | Convert packed double-precision floating-point values to packed single-precision floating-point values

```
CVTPD2PS xmm(n), xmm(n)
CVTPD2PS xmm(n), double(2)
```

Converte dois valores _double_ do operando fonte (segundo) em dois valores _float_ no operando destino (primeiro).

### CVTSS2SD | Convert scalar single-precision floating-point value to scalar double-precision floating-point value

```
CVTSS2SD xmm(n), xmm(n)
CVTSS2SD xmm(n), float(1)
```

Converte um valor _float_ do operando fonte (segundo) em um valor _double_ no operando destino (primeiro).

### CVTSD2SS | Convert scalar double-precision floating-point value to scalar single-precision floating-point value

```
CVTSD2SS xmm(n), xmm(n)
CVTSD2SS xmm(n), double(1)
```

Converte um valor _double_ do operando fonte (segundo) em um valor _float_ no operando destino (primeiro).

## Conversão entre double e inteiro

### CVTPD2DQ/CVTTPD2DQ | Convert (with truncation) packed double-precision floating-point values to packed doubleword integers

```
CVTPD2DQ xmm(n), xmm(n)
CVTPD2DQ xmm(n), double(2)


CVTTPD2DQ xmm(n), xmm(n)
CVTTPD2DQ xmm(n), double(2)
```

Converte os dois _doubles_ no operando fonte para dois inteiros sinalizados de 32-bit no operando destino. A instrução CVTPD2DQ faz o arredondamento normal do valor enquanto CVTTPD2DQ trunca ele.

### CVTDQ2PD | Convert packed doubleword integers to packed double-precision floating-point values

```
CVTDQ2PD xmm(n), xmm(n)
CVTDQ2PD xmm(n), dword(2)
```

Converte os dois inteiros sinalizados de 32-bit no operando fonte para dois _doubles_ no operando destino.

### CVTSD2SI/CVTTSD2SI | Convert scalar double-precision floating-point value to doubleword integer

```
CVTSD2SI reg32/64, xmm(n)
CVTSD2SI reg32/64, double(1)

CVTTSD2SI reg32/64, xmm(n)
CVTTSD2SI reg32/64, double(1)
```

CVTSD2SI converte o valor _double_ no operando fonte em inteiro de 32-bit sinalizado, e armazena o valor no registrador de propósito geral do operando destino. O registrador destino também pode ser um registrador de 64-bit onde nesse caso o valor sofrerá extensão de sinal ([sign extension](https://en.wikipedia.org/wiki/Sign_extension)).

CVTTSD2SI faz a mesma coisa porém truncando o valor.

### CVTSI2SD | Convert doubleword integer to scalar double-precision floating-point value

```
CVTSI2SD xmm(n), reg32/64
CVTSI2SD xmm(n), dword(1)
CVTSI2SD xmm(n), qword(1)
```

Converte o valor inteiro sinalizado de 32 ou 64 bits do operando fonte e armazena como um _double_ no operando destino.

## Conversão entre float e inteiro

### CVTPS2DQ/CVTTPS2DQ | Convert (with truncation) packed single-precision floating-point values to packed doubleword integers

```
CVTPS2DQ xmm(n), xmm(n)
CVTPS2DQ xmm(n), float(4)


CVTTPS2DQ xmm(n), xmm(n)
CVTTPS2DQ xmm(n), float(4)
```

Converte quatro _floats_ do operando fonte em quatro inteiros sinalizados de 32-bit no operando destino. A instrução CVTPS2DQ faz o arredondamento normal dos valores enquanto CVTTPS2DQ trunca eles.

### CVTDQ2PS | Convert packed doubleword integers to packed single-precision floating-point values

```
CVTDQ2PS xmm(n), xmm(n)
CVTDQ2PS xmm(n), dword(4)
```

Converte quatro inteiros sinalizados de 32-bit no operando fonte para quatro _floats_ no operando destino.

### CVTSS2SI/CVTTSS2SI | Convert scalar single-precision floating-point value to doubleword integer

```
CVTSS2SI reg32/64, xmm(n)
CVTSS2SI reg32/64, float(1)


CVTTSS2SI reg32/64, xmm(n)
CVTTSS2SI reg32/64, float(1)
```

CVTSS2SI converte o valor _float_ no operando fonte em inteiro de 32-bit sinalizado, e armazena o valor no registrador de propósito geral do operando destino. O registrador destino também pode ser um registrador de 64-bit onde nesse caso o valor sofrerá extensão de sinal ([sign extension](https://en.wikipedia.org/wiki/Sign_extension)).

A instrução CVTTSS2SI faz a mesma coisa porém truncando o valor.

### CVTSI2SS | Convert doubleword integer to scalar single-precision floating-point value

```
CVTSI2SS xmm(n), reg32/64
CVTSI2SS xmm(n), dword(1)
CVTSI2SS xmm(n), qword(1)
```

Converte o valor inteiro sinalizado de 32 ou 64 bits do operando fonte e armazena como um _float_ no operando destino.
