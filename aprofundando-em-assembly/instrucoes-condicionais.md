---
description: Entendendo as instruções condicionais e as status flags.
---

# Instruções condicionais

As instruções condicionais basicamente avaliam as _status flags_ para executar uma operação apenas se a condição for atendida. Existem condições que testam o valor de mais de uma _flag_ em combinação para casos diferentes.

A nomenclatura de escrita de uma instrução condicional é o seu nome seguido de um 'cc' que é sigla para _conditional code_. Abaixo uma tabela de códigos condicionais válidos para as instruções `CMOVcc`, `SETcc` e `Jcc`:

{% hint style="info" %}
Os termos "abaixo" (_below_) e "acima" (_above_) usados na descrição se referem a verificação de um valor numérico não-sinalizado. Enquanto "maior" e "menor" é usado para se referir a um valor numérico sinalizado.
{% endhint %}

| cc  | Descrição (inglês \| português)                  | Condição       |
| --- | ------------------------------------------------ | -------------- |
| A   | if Above \| se acima                             | CF=0 e ZF=0    |
| AE  | if Above or Equal \| se acima ou igual           | CF=0           |
| B   | if Below \| se abaixo                            | CF=1           |
| BE  | if Below or Equal \| se acima ou igual           | CF=1 ou ZF=1   |
| C   | if Carry \| se carry flag estiver setada         | CF=1           |
| E   | if Equal \| se igual                             | ZF=1           |
| G   | if Greater \| se maior                           | ZF=0 e SF=OF   |
| GE  | if Greater or Equal \| se maior ou igual         | SF=OF          |
| L   | if Less \| se menor                              | SF!=OF         |
| LE  | if Less or Equal \| se menor ou igual            | ZF=1 ou SF!=OF |
| NA  | if Not Above \| se não acima                     | CF=1 ou ZF=1   |
| NAE | if Not Above or Equal \| se não acima ou igual   | CF=1           |
| NB  | if Not Below \| se não abaixo                    | CF=0           |
| NBE | if Not Below or Equal \| se não abaixou ou igual | CF=0 e ZF=0    |
| NC  | if Not Carry \| se carry flag não estiver setada | CF=0           |
| NE  | if Not Equal \| se não igual                     | ZF=0           |
| NG  | if Not Greater \| se não maior                   | ZF=1 ou SF!=OF |
| NGE | if Not Greater or Equal \|se não maior ou igual  | SF!=OF         |
| NL  | if Not Less \| se não menor                      | SF=OF          |
| NLE | if Not Less or Equal \| se não menor ou igual    | ZF=0 e SF=OF   |
| NO  | if Not Overflow \| se não setado overflow flag   | OF=0           |
| NP  | if Not Parity \| se não setado parity flag       | PF=0           |
| NS  | if Not Sign \| se não setado sign flag           | SF=0           |
| NZ  | if Not Zero \| se não setado zero flag           | ZF=0           |
| O   | if Overflow \| se setado overflow flag           | OF=1           |
| P   | if Parity \| se setado parity flag               | PF=1           |
| PE  | if Parity Even \| se parity indica par           | PF=1           |
| PO  | if Parity Odd \| se parity indicar ímpar         | PF=0           |
| S   | if Sign \| se setado sign flag                   | SF=1           |
| Z   | if Zero \| se setado zero flag                   | ZF=1           |

Exemplo:

```nasm
cmp rax, rbx
jnz nao_igual  ; Salta se RAX e RBX não forem iguais
```

Repare como alguns **cc** têm a mesma condição, como é o caso de NE e NZ. Portanto `JNE` e `JNZ` são exatamente a mesma instrução no código de máquina, somente mudando no Assembly.

### JCXZ e JECXZ

Além das condições acima existem mais três `Jcc` que testam o valor do registrador CX, ECX e RCX respectivamente.

| Jcc   | Descrição (inglês \| português)                     | Condição |
| ----- | --------------------------------------------------- | -------- |
| JCXZ  | Jump if CX is zero \| pula se CX for igual a zero   | CX=0     |
| JECXZ | Jump if ECX is zero \|pula se ECX for igual a zero  | ECX=0    |
| JRCXZ | Jump if RCX is zero \| pula se RCX for igual a zero | RCX=0    |

A última instrução, obviamente, somente existe em submodo de 64-bit. Enquanto `JCXZ` não existe em 64-bit.

No código de máquina o opcode dessa instrução é **0xE3** e a alternância entre o tamanho do registrador é feita de acordo com o atributo _address-size_, sendo modificado pelo prefixo **0x67**.
