---
description: Ainda não vimos tudo
---

# Call e Ret

Quando se trata de chamadas de procedimentos existem dois conceitos relacionados ao endereço deste procedimento. Existem chamadas "próximas" \(_near_\) e "distantes" \(_far_\).  
Enquanto no _near_ `call` nós apenas especificamos o _offset_ do endereço, no _far_ `call` nós também especificamos o segmento.

O outro conceito é de endereço "relativo" \(_relative_\) e "absoluto" \(_absolute_\).

### Tamanho do offset

O tamanho que o offset do endereço deve ter acompanha a largura do barramento interno. Então se estamos em _real mode_ \(16 bit\), por padrão o _offset_ deve ser de 16 bit.  
Ou seja, basicamente o mesmo tamanho do _Instruction Pointer_.

### Near Relative Call

```text
call rel16/rel32
```

Esta é a `call` que já usamos, não tem segredo. Ela basicamente recebe um número negativo ou positivo indicando o número de bytes que devem ser desviados. Veja da seguinte forma:

```text
Instruction_Pointer = Instruction_Pointer + operand
```

A matemática básica nos diz que "mais com menos é menos", ou seja, se o operando for negativo essa soma resultará em uma subtração.

#### Onde está RIP?

Existe um detalhe bem simples porém importante para conseguir lidar com endereços relativos corretamente. Quando o processador for executar a instrução, o _Instruction Pointer_ já estará apontando para a instrução seguinte.  
Isso faz toda a diferença porque desvios de fluxo para trás precisam contar os bytes da própria instrução em si.  
Claro que este cálculo não é feito por nós e sim pelo assembler, mas é importante saber.  
Ah, e lembra do símbolo **$** que eu falei que o nasm expande para o endereço da instrução atual?  
Veja que ele não coincide com o valor de RIP, cujo o mesmo já está apontando para a instrução seguinte.  
Por exemplo, poderíamos fazer uma chamada na própria instrução gerando um loop "infinito" usando a sintaxe:

```text
call $
```

### Near Absolute Call

```text
call r/m
```

Diferente da chamada relativa que indica um número de bytes a serem somados com RIP, numa chamada absoluta você passa o endereço exato de onde você quer fazer a chamada.  
Você pode experimentar fazer uma chamada assim:

```text
mov  rax, rotulo
call rax
```

Se você passar `rotulo` para a `call` diretamente, você estará fazendo uma chamada relativa porque deste jeito você estará passando um valor imediato. E a única `call` que recebe valor imediato é a de endereço relativo, por isso o nasm passa o endereço relativo daquele rótulo.  
Mas ao definir o endereço do rótulo para um registrador ou memória, o assembler irá passar o endereço absoluto dele.

É importante entender que tipo de operando cada instrução recebe para evitar se confundir sobre como o assembler irá montar a instrução.  
E sim, saber como a instrução é montada em código de máquina é muitas vezes importante.

### Far Call

```text
call seg16:off16   ; Em 16-bit
call seg16:off32   ; Em 32-bit

call mem16:16  ; Em 16-bit
call mem16:32  ; Em 32-bit
call mem16:64  ; Em 64-bit
```

As chamadas far são todas absolutas e recebem no operando um valor seguindo o formato de especificar um _offset_ seguido do segmento de 16-bit.  
No nasm, um valor imediato pode ser passado da seguinte forma:

```text
call 0x1234:0xabcdef99
```

Onde o valor a esquerda especifica o segmento e o da direita o _offset_. Detalhe que esta instrução não é suportada em 64-bit.

O segundo tipo de _far_ `call`, suportado em 64-bit, é o que recebe como operando um valor  na memória. Mas perceba que temos um _near_ `call` que recebe o mesmo tipo de argumento, não é mesmo?  
Por padrão o nasm irá montar as instruções como _near_ e não _far_. Mas você pode evitar essa ambiguidade explicitando com _keywords_ do nasm, que são bem intuitivas. Veja:

```text
call [rbx]       ; Próximo e absoluto
call near [rbx]  ; Próximo e absoluto
call far [rbx]   ; Distante
```

O _near_ espera o endereço do _offset_ em memória, não tem segredo. Mas o _far_ espera o _offset_ seguido do segmento.  
Em um sistema de 32-bit vamos supor que nosso procedimento está no segmento 0xaaaa e no _offset_ 0xbbbb1111. Em memória o valor precisa estar assim:

```text
11 11 bb bb aa aa
```

Lembrando que x86 é _little-endian_.

Basicamente o _far_ `call` modifica o valor de CS e IP ao mesmo tempo, enquanto o _near_ `call` apenas modifica o valor de IP.

{% hint style="info" %}
No código de máquina, a diferença entre o _far_ e o _near_ `call` que usam o operando em memória está no campo REG do byte ModR/M. O _near_ tem o valor 2 e o _far_ tem o valor 3. O opcode é `FF`.  
Se você não entendeu isso aqui, não se preocupa com isso...
{% endhint %}

### Ret

```text
ret(f/n)
ret(f/n) imm16
```

Como talvez você já tenha reparado intuitivamente, a chamada _far_ também preserva o valor de CS na _stack_ e não apenas o valor de IP. \(lembrando que IP já estaria apontando para a instrução seguinte...\)  
Por isso a instrução `ret` também precisa ser diferente. Ao invés de apenas ler o _offset_ na _stack_ ela precisa ler o segmento também, assim modificando CS e IP do mesmo jeito que o `call`.

Repetindo que o nasm por padrão irá montar as instruções como _near_ então precisamos especificar para o nasm, em um procedimento que deve ser chamado como _far_, __que queremos usar um `ret` _far_.  
Para isso podemos simplesmente adicionar um sufixo 'n' para especificar como _near_, que já é o padrão, ou o sufixo 'f' para especificar como _far_.  
Ficando:

```text
retf  ; Usado em procedimentos que devem ser chamados com far call
```

Existe também uma outra opção de instrução `ret` que recebe como operando um valor imediato de 16-bit que especifica um número de bytes a serem desempilhados da _stack_.  
Basicamente o que ele faz é somar o valor de SP com esse número, porque como sabemos a pilha cresce "para baixo".  
Ou seja, se subtraímos valor em SP estamos fazendo a pilha crescer. Se somamos, estamos fazendo ela diminuir.  
Por exemplo, podemos escrever em pseudo-código a instrução `retf 12` da seguinte forma:

{% code title="pseudo.c" %}
```c
RIP = pop();
CS  = pop();
RSP = RSP + 12;
```
{% endcode %}

