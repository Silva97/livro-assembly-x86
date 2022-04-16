---
description: Entendendo o código de máquina x86-64
---

# Código de máquina

O famigerado **código de máquina** (também chamado de **linguagem de máquina**), popularmente conhecido como "zeros e uns", são as instruções que o processador interpreta e executa. São basicamente números onde o processador decodifica esses números afim de executar determinadas operações identificadas pelas instruções.

Acho que boa parte das pessoas da área da computação sabem que processadores de computadores digitais funcionam com sinais elétricos com duas tensões diferentes: Uma alta (lá pelos 3v, mas pode variar de acordo com o processador) e uma baixa (perto de 0v), onde a tensão alta representa o 1 e a tensão baixa representa o 0.

Mas comumente é só isso o que as pessoas sabem sobre código de máquina. O objetivo deste capítulo é dar uma noção aprofundada de como funciona o código de máquina da arquitetura x86-64.

{% hint style="info" %}
Cada arquitetura de processador (vulgo ISA, _Instruction Set Architecture_) têm um código de máquina distinto. Portanto as informações aqui são válidas para código de máquina x86 e x86-64. ARM, RISC-V etc. contém código de máquina que funciona de um jeito completamente diferente.
{% endhint %}

### Representação textual

Antes de mais nada um pré-aviso: Sei que é romântico quando se fala de código de máquina meter um monte de zeros e uns (como: `10110100010`). Mas na vida real **ninguém** representa textualmente código de máquina em binário. Isso é normalmente feito em manuais ou ferramentas como _disassemblers_ e _debuggers_ usando **hexadecimal**.

Então ao pensar em código de máquina não pense nisso `10110100 00001110` mas sim nisso `B4 0E`. Você é humano, pense como tal.

### Ferramentas

Comecei a desenvolver uma ferramenta exclusivamente para ser usada como auxílio para esse capítulo. Eu a chamei de **x86-visualizer** e seu intuito é você escrever uma instrução em Assembly e ela lhe exibir o código de máquina dividido em seus campos, assim facilitando o entendimento.

A ferramenta não está concluída então poucas instruções irão funcionar, todavia sugiro seu uso durante a leitura do capítulo afim de facilitar o entendimento da codificação das instruções.

Acesse o repositório dela aqui:

* [https://github.com/Silva97/x86-visualizer](https://github.com/Silva97/x86-visualizer)

Também sugiro usar o **ndisasm** afim de fazer experimentações. Ele é um _disassembler_ que vem junto com o **nasm** e [já foi utilizado anteriormente no livro](../../aprofundando-em-assembly/atributos.md).
