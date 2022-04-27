---
description: Entendendo o prefixo REX no x86-64.
---

# Prefixo REX

Como eu mencionei antes esse prefixo só existe no modo de 64-bit e ele é necessário para usar operandos de 64-bit. Esse prefixo não é um byte específico mas sim todos os bytes entre `40` e `4F`. Isso porque os últimos 4 bits do prefixo são campos distintos, mas os 4 bits mais significativos do prefixo REX sempre tem o valor fixo de `0100`.

Observe as figuras tiradas dos manuais da Intel:

![](../../.gitbook/assets/Captura\_de\_tela\_de\_2022-04-04\_21-31-47.png)



Em modo de 16-bit e 32-bit há 8 registradores de propósito geral, mas em 64-bit há 16 registradores de propósito geral. Como eu mencionei antes os campos que especificam os registradores por códigos contém somente 3 bits de tamanho, daí só é possível especificar 8 registradores distintos.

Mas alguns bits do prefixo REX são usados para estender os tamanhos desses campos em 1 bit, assim permitindo especificar até 16 registradores distintos ou 16 modos de endereçamento distintos. Cada bit do prefixo REX é identificado por uma letra e é comumente referido como no formato `REX.B` que seria o bit `B` (o menos significativo) do prefixo.

#### **REX.B (bit 0)**

Em instruções cujo a codificação do registrador faz parte do opcode, ele é usado para estender o campo de registrador. Onde ele se torna o bit mais significativo do valor.

Em instruções com ModR/M (sem SIB) ele estende o campo R/M como o bit mais significativo.

Em instruções com SIB ele estende o campo **Base** como o bit mais significativo.

#### **REX.X (bit 1)**

Estende o campo **Index** do SIB como o bit mais significativo.

#### **REX.R (bit 2)**

Estende o campo REG do byte ModR/M como o bit mais significativo.

#### **REX.W (bit 3)**

Se ligado a instrução usa operandos de 64-bit, onde por padrão os operandos são de 32-bit.
