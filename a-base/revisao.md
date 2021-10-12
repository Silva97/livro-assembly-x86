---
description: Entenda tudo o que viu aqui
---

# Revisão

As instruções de Assembly por si só na verdade é bem simples, como já vimos antes a sintaxe de uma instrução é bem fácil e entender o que ela faz também não é o maior segredo do mundo.

Porém como também já vimos, para de fato ter conhecimento adequado da linguagem é necessário aprender muita coisa e talvez esse conhecimento variado tenha ficado disperso na sua mente (e olha que só aprendemos um pouco). A ideia desse tópico é juntar tudo e mostrar como e porque está relacionado à Assembly.

### O que estamos fazendo?

Programar em uma linguagem de baixo nível como Assembly não é a mesma coisa de programar em uma linguagem de alto nível como C. Ao programar em Assembly estamos escrevendo **diretamente** as instruções que serão executadas pelo processador.

Não apenas isso como também estamos organizando todo o formato do arquivo de acordo com o formato final que queremos executar. Então é importante entender duas coisas, antes de mais nada: A arquitetura para a qual estamos programando e o formato de arquivo que queremos escrever.

A arquitetura é a x86, como já sabemos. E um código que irá trabalhar com a linguagem C é compilado para um arquivo objeto. Por isso estudamos os conceitos básicos da arquitetura x86 propriamente dita, e também estudamos um pouco do arquivo objeto.

Sem saber o que são seções, o que á _symbol table_ etc. não dá para entender o que se está fazendo. **Por que** o código em C consegue acessar um rótulo no meu código em Assembly? **Por que** dados despejados em `.data` são chamados de variáveis e os em `.text` são chamados de código? **Por que** dados em `.rodata` não podem ser modificados e são chamados de constantes? **Por que** "isso" é considerado uma função e "isso" uma variável? Os dois não são símbolos?

Ao programar em Assembly nós não estamos apenas escrevendo as instruções que o processador irá executar, estamos também construindo todo o arquivo binário final manualmente.Felizmente o NASM facilita nossa vida ao formatar o arquivo binário para o formato desejado, é a tal da opção **-f elf64** ou **-f win64** que passamos na linha de comando. Mas mesmo assim temos que dar informações para o NASM sobre o que fica aonde.

### Por que estamos fazendo isso?

Em uma linguagem de alto nível todos esse conceitos relacionados ao formato do arquivo binário e da arquitetura do processador são abstraídos. Já em uma linguagem de baixo nível, esses conceitos tem muito pouca (ou nenhuma) abstração e precisamos lidar com eles manualmente.

Isso é necessário porque estamos escrevendo diretamente as instruções que o _hardware_, o processador, irá executar. E para poder se comunicar com o processador precisamos entender o que ele está fazendo.

Imagine tentar instruir um funcionário de uma empresa de entregas exatamente como ele deve organizar a carga e como ele deve entregá-la, porém você não sabe o que é a carga e nem para quem ela deve ser entregue. Impossível, né?

### Isso é útil por quê?

Estudar Assembly não é só decorar instruções e o que elas fazem, isso é fácil até demais. Estudar Assembly é estudar a arquitetura, o formato do executável, como o executável funciona, convenções de chamadas, características do sistema operacional, características do hardware etc... Ah, e estudar as instruções também. Junte tudo isso e você terá um belo conhecimento para entender como um software funciona na prática.

Já citei antes porque estudar Assembly é útil na introdução do livro. Mas só decorar as instruções não é útil por si só, a questão é todo o resto que você irá aprender ao estudar Assembly.

### O que é um código fonte em Assembly?

Como já vimos o assembler é bem mais complexo do que simplesmente converter as instruções que você escreve em código de máquina. Ele tem diretivas, pseudo-instruções e pré-processamento de código para formatar o código em Assembly e lhe dar mais poder na programação.

Ele também formata o código de saída para um formato especificado e nos permite escolher o modo de compilação das instruções de 64, 32 ou 16 bits.

Ou seja, um código fonte em Assembly não é apenas instruções mas também diretivas para a formatação do arquivo binário que será feita pelo assembler. Que é muito diferente de uma linguagem de alto nível como C que contém apenas instruções e todo o resto fica abstraído como se nem existisse.
