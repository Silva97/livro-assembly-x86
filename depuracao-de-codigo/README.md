---
description: Aprendendo a depurar código em nível de Assembly
---

# Depuração de código

O termo depuração (_debugging_) é usado na área da computação para se referir ao ato de procurar e corrigir falhas (_bugs_) em softwares. A principal ferramenta, embora não única, usada para essa tarefa é um tipo de software conhecimento como depurador (_debugger_). Essa ferramenta basicamente dá ao programador a possibilidade de controlar a execução de um programa enquanto ele pode ver informações sobre o processo em tempo de execução.

Existem depuradores que meramente exibem o código-fonte do programa e o programador acompanha a execução do código vendo o código-fonte do projeto. Mas existe uma categoria de depurador que exibe o _disassembly_ do código do programa e o programador é capaz de ver a execução do código acompanhando as instruções em Assembly.

Este capítulo tem por objetivo dar uma noção básica de como depuradores funcionam e ensinar a usar algumas ferramentas para depuração de código.

## Ferramentas

O conteúdo será principalmente baseado em ferramentas sendo utilizadas em ambiente Linux, porém a maior parte do conteúdo é reaproveitável no Windows.

Códigos de exemplo serão escritos em C e compilados com o GCC, bem como alguns serão escritos diretamente em Assembly e usando o _assembler_ NASM.
