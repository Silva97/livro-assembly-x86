---
description: Campo immediate na instrução do código de máquina.
---

# Immediate

O campo _immediate_ (valor "imediato") pode ter 1, 2, ou 4 bytes de tamanho. Ele é o operando numérico presente em algumas instruções. Exemplo:

```nasm
mov eax, 0x11223344
```

Essa instrução em código de máquina fica: `B8 44 33 22 11`

Onde `B8` é o _opcode_ da instrução e `44 33 22 11` o valor imediato (`0x11223344`). Lembrando que a arquitetura x86 é _little-endian_, portanto o valor imediato fica em _little-endian_ na instrução.

O tamanho desse campo é definido pelo atributo **operand-size**, portanto ao usar o prefixo `66` o seu tamanho pode alternar na instrução entre 16-bit e 32-bit. Sobre instruções com operandos de 8-bit, como `mov al, 123`, existem _opcodes_ específicos para operandos nesse tamanho portanto o prefixo não é usado nessas instruções. E obrigatoriamente o _immediate_ terá 8-bit de tamanho.

Outros dois exemplos seriam `mov ax, 0x1122` e `mov al, 0x11`. Onde o primeiro tem o código de máquina `66 B8 22 11` em modo de 32-bit, e em modo de 16-bit fica igual só que sem o prefixo `66`.

Já a segunda instrução terá o código de máquina `B0 11` em qualquer modo de operação, já que ela independe do **operand-size**.
