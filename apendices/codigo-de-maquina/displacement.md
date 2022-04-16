---
description: Campo displacement na instrução do código de máquina.
---

# Displacement

O _displacement_ (deslocamento) é um valor numérico de 1, 2 ou 4 bytes de tamanho que também faz parte da instrução assim como o valor imediato.

{% hint style="info" %}
Em modo de 32-bit ou 64-bit, o _displacement_ pode ser de 1 ou 4 bytes de tamanho. Em modo de 16-bit pode ser de 1 ou 2 bytes de tamanho.
{% endhint %}

Ele é um valor numérico que é somado ao [endereçamento](../../a-base/enderecamento.md) definido pelo byte ModR/M. Se esse campo está presente ou não na instrução, bem como seu tamanho, é definido no byte ModR/M.

Exemplo:

![Print do x86-visualizer.](<../../.gitbook/assets/Captura de tela de 2022-04-16 12-51-02.png>)

Onde o valor `0x11223344` na instrução `mov eax, [ebx + 0x11223344]` é o _displacement_ da instrução.
