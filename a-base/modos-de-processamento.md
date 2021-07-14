---
description: Entendendo os diversos modos de processamento presentes em processadores x86
---

# Modos de processamento

Como já explicado a arquitetura x86 foi uma evolução ao longo dos anos e sempre mantendo compatibilidade com os processadores anteriores. Mas código de 16, 32 e 64 bit são demasiadamente diferentes e boa parte das instruções não são equivalentes o que teoricamente faria com que, por exemplo, código de 32 bit fosse impossível de rodar em um processador x86-64. Mas é aí que entra os modos de processamento.

Um processador x86-64 consegue executar código de versões anteriores simplesmente trocando o modo de processamento. Cada modo faz com que o processador funcione de maneira um tanto quanto diferente, fazendo com que as instruções executadas também tenham resultados diferentes.

Ou seja, lá no 8086 seria como se só existisse o modo de 16 bit. Com a chegada dos processadores de 32 bit na verdade simplesmente foi adicionado um novo modo de processamento aos processadores que seria o modo de 32 bit. E o mesmo aconteceu com a chegada dos processadores x86-64 que basicamente adiciona um modo de processamento de 64 bit. É claro que além dos modos de processamento novos também surgem novas tecnologias e novas instruções, mas o modo de processamento anterior fica intacto e por isso se tem compatibilidade com os processadores anteriores.

Podemos dizer que existem três modos de processamento principais:

| Modo de processamento | Largura do barramento interno |
| :--- | :--- |
| Real mode / Modo real | 16 bit |
| Protected mode / Modo protegido | 32 bit |
| 64-bit submode / Submodo de 64-bit | 64 bit |

### Barramento interno

Os tais "bit" que são muito conhecidos mas pouco entendido, na verdade é simplesmente uma referência a largura do barramento interno do processador quando ele está em determinado modo de processamento. A largura do barramento interno do processador nada mais é que o tamanho padrão de dados que ele pode processar de uma única vez.

Imagine uma enorme via com 16 faixas e no final dela um pedágio, isso significa que 16 carros serão atendidos por vez no pedágio. Se é necessário atender 32 carros então será necessário duas vezes para atender todos os carros, já que apenas 16 podem ser atendidos de uma única vez. A largura de um barramento nada mais é que uma "via de bits", quanto mais largo mais informação pode ser enviada de uma única vez. O que teoricamente aumenta a eficiência.

No caso do barramento interno do processador seria a "via de bits" que o processador usa em todo o seu sistema interno, desconsiderando a comunicação com o _hardware_ externo que é feita pelo barramento externo e não necessariamente tem o mesmo tamanho do barramento interno.

{% hint style="info" %}
Também existe o barramento de endereço, mas não vamos abordar isso agora.
{% endhint %}

### Mais modos de processamento

Pelo que nós vimos acima então na verdade um "sistema operacional de 64 bit" nada mais é que um sistema operacional que executa em submodo de 64-bit. Ah, mas aí fica a pergunta:

> Se está rodando em 64 bit como é possível executar código de 32 bit?

Isso é possível porque existem mais modos de processamento do que os que eu já mencionei. Reparou que eu disse "submodo" de 64-bit? É porque na verdade o 64-bit não é um modo principal mas sim um submodo. A hierarquia de modos de processamento de um processador Intel64 ficaria da seguinte forma:

* Real mode \(16 bit\)
* Protected mode \(32 bit\)
* SMM \(não vamos falar deste modo, mas ele existe\)
* IA-32e
  * 64-bit \(64 bit\)
  * Compatibility mode \(32 bit\)

O modo **IA-32e** é uma adição dos processadores x86-64. Repare que ele tem outro submodo chamado "_compatibility mode_", ou em português, "modo de compatibilidade".

{% hint style="warning" %}
Não confundir com o modo de compatibilidade do Windows, ali é uma coisa diferente que leva o mesmo nome.
{% endhint %}

O modo de compatibilidade serve para obter compatibilidade com a arquitetura IA-32. Um sistema operacional pode setar para que código de apenas determinado segmento na memória rode nesse modo, permitindo assim que ele execute código de 32 e 64 bit paralelamente \(supondo que o processador esteja em modo IA-32e\). Por isso que seu Debian de 64 bit consegue rodar softwares de 32 bit, assim como o seu Windows 10 de 64 bit também consegue.

### Virtual-8086

Lembra que o antigo Windows XP de 32 bit era capaz de rodar programas de 16 bit do MS-DOS?  
Isto era possível devido ao modo Virtual-8086 que, de maneira parecida com o _compatibility mode_, permite executar código de 16 bit enquanto o processador está em _protected mode_. Nos processadores atuais o Virtual-8086 não é um submodo de processamento do _protected mode_ mas sim um atributo que pode ser setado enquanto o processador está executando nesse modo.

{% hint style="info" %}
Repare que rodando em _compatibility mode_ não é possível usar o modo Virtual-8086. É por isso que o Windows XP de 32 bit conseguia rodar programas do MS-DOS mas o XP de 64 bit não.
{% endhint %}

