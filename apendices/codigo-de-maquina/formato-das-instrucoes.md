---
description: O formato das instruções do código de máquina.
---

# Formato das instruções

### CISC

Primeira coisa que a gente precisa saber é que a arquitetura x86-64 é CISC (_Complex Instruction Set Computer_), ou seja uma arquitetura que contém um conjunto complexo de instruções.

O que significa na prática que a arquitetura contém muitas instruções consideradas "complexas", que efetuam muitas operações de uma vez. Por exemplo a instrução `rep movsb` faz um bocado de coisas:

1. Copia o valor em `DS:ESI` para `ES:EDI`.
2. Incrementa o valor de ESI.
3. Incrementa o valor de EDI.
4. Decrementa o valor de ECX.
5. Verifica se o valor de ECX é zero. Se for finaliza o _loop_.

Tudo isso em apenas uma instrução.

### Formato

Esse é o formato de uma instrução do código de máquina da arquitetura segundo os manuais da Intel:

![Figura retirada dos manuais da Intel. Apêndice B do volume 2.](../../.gitbook/assets/Captura\_de\_tela\_de\_2022-04-03\_14-17-59.png)



**Legacy prefixes**: são prefixos que existem desde o x86, alguns até mesmo desde o 8086. Por isso são chamados de "_legacy_" (legados).

**REX prefix**: é um prefixo novo existente somente no modo de 64-bit e adicionado em processadores x86-64.

**Opcode**: abreviação para _operation code_ (código de operação), é um valor numérico (de 1 a 3 bytes de tamanho) que identifica qual operação o processador deve executar. Desde mover valores, subtrair, somar, calcular a raiz quadrada, modificar o valor de um registrador etc.

**ModR/M**: é um byte na instrução que não está presente em todas elas. Explico em detalhes depois mas ele serve para definir o modo de endereçamento e/ou qual registrador é usado na operação. Por isso o R/M, que é uma abreviação para _**R**egister/**M**emory_.

**SIB**: dependendo do modo de endereçamento definido em **ModR/M**, o byte SIB pode ser usado. Ele define três valores:

* **Scale** (2 bits): determina um fator de "escala" (1, 2, 4 ou 8) que irá multiplicar o valor do **index**.
* **Index** (3 bits): define o registrador que será usado como índice.
* **Base** (3 bits): define o registrador que será usado como base. Na prática o cálculo do endereçamento é feito como na seguinte pseudo-expressão:

```
address = base + index * scale
```

**Displacement**: é um valor numérico de 1, 2 ou 4 bytes de tamanho que é somado ao endereçamento definido por ModR/M. Nem todo modo de endereçamento definido por ModR/M usa o _displacement_, então nem sempre ele está presente em uma instrução com operando na memória.

**Immediate**: é um valor numérico de 1, 2 ou 4 bytes de tamanho usado em algumas operações que usam um operando imediato. Por exemplo `mov ah, 0x0E`, onde o número `0x0E` (14 em decimal) é o valor **immediate** na instrução.

Inclusive a instrução `B4 0E` que [mencionei anteriormente](./#representacao-textual) é a `mov ah, 0x0E`. Onde `B4` é o **opcode** (de 1 byte) e `0E` o **immediate** (de 1 byte também).

{% hint style="info" %}
Uma instrução na arquitetura x86 pode ter de 1 até 15 bytes de tamanho. E caso ainda não tenha ficado claro: sim, instruções na arquitetura x86 têm tamanhos variados.
{% endhint %}
