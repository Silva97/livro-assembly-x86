---
description: Entendendo detalhadamente as instruções CALL e RET.
---

# CALL e RET

Quando se trata de chamadas de procedimentos existem dois conceitos relacionados ao endereço deste procedimento.

O primeiro conceito é que existem chamadas "próximas" (_near_) e "distantes" (_far_). Enquanto no _near_ `call` nós apenas especificamos o _offset_ do endereço, no _far_ `call` nós também especificamos o segmento.

O outro conceito é o de endereço "relativo" (_relative_) e "absoluto" (_absolute_), que também se aplicam para saltos (_jumps_). Onde um endereço relativo é basicamente um número **sinalizado** que será somado à RIP quando o desvio de fluxo ocorrer. Enquanto o endereço absoluto é um endereço exato que será escrito no registrador RIP.

### Tamanho do offset

O tamanho que o _offset_ do endereço deve ter acompanha a largura do barramento interno. Então se estamos em _real mode_ (16 bit), por padrão o _offset_ deve ser de 16-bit. Ou seja, basicamente o mesmo tamanho do _Instruction Pointer_.

### Near relative call

```
call rel16/rel32
```

Essa é a `call` que já usamos, não tem segredo. Ela basicamente recebe um número negativo ou positivo indicando o número de bytes que devem ser desviados. Veja da seguinte forma:

```
Instruction_Pointer = Instruction_Pointer + operand
```

A matemática básica nos diz que "mais com menos é menos", ou seja, se o operando for negativo essa soma resultará em uma subtração.

#### Onde está RIP?

Existe um detalhe bem simples porém importante para conseguir lidar com endereços relativos corretamente. Quando o processador for executar a instrução o _Instruction Pointer_ já estará apontando para a instrução seguinte. Ou seja desvios de fluxo para trás precisam contar os bytes da própria instrução em si, enquanto os para frente começam contando em zero que já é a instrução seguinte na memória.

Claro que esse cálculo não é feito por nós e sim pelo assembler, mas é importante saber. Ah, e lembra do símbolo `$` que eu falei que o NASM expande para o endereço da instrução atual? Veja que ele não coincide com o valor de RIP, cujo o mesmo já está apontando para a instrução seguinte.

Por exemplo poderíamos fazer uma chamada na própria instrução gerando um loop "infinito" usando a sintaxe:

```nasm
bits 64

call $
```

Experimente ver com o ndisasm como essa instrução fica em código de máquina:

![](<../.gitbook/assets/image (8).png>)

O primeiro byte (`0xE8`) é o opcode da instrução, que é o byte do código de máquina que identifica a instrução que será executada. Os bytes posteriores são o operando imediato (em _little-endian_). Repare que o endereço relativo está como `0xFFFFFFFB` que equivale a `-5` em decimal.

### Near absolute call

```
call r/m
```

Diferente da chamada relativa que indica um número de bytes a serem somados com RIP, numa chamada absoluta você passa o endereço exato de onde você quer fazer a chamada. Você pode experimentar fazer uma chamada assim:

```nasm
mov  rax, rotulo
call rax
```

Se você passar `rotulo` para a `call` diretamente você estará fazendo uma chamada relativa porque desse jeito você estará passando um operando imediato. E a única `call` que recebe valor imediato é a de endereço relativo, por isso o NASM passa o endereço relativo daquele rótulo. Mas ao definir o endereço do rótulo para um registrador ou memória o assembler irá passar o endereço absoluto dele.

É importante entender que tipo de operando cada instrução recebe para evitar se confundir sobre como o assembler irá montar a instrução. E sim, saber como a instrução é montada em código de máquina é muitas vezes importante.

### Far call

```
call seg16:off16   ; Em 16-bit
call seg16:off32   ; Em 32-bit

call mem16:16  ; Em 16-bit
call mem16:32  ; Em 32-bit
call mem16:64  ; Em 64-bit
```

As chamadas _far_ (distante) são todas absolutas e recebem no operando um valor seguindo o formato de especificar um _offset_ seguido do segmento de 16-bit. No NASM um valor imediato pode ser passado da seguinte forma:

```nasm
call 0x1234:0xabcdef99
```

Onde o valor à esquerda especifica o segmento e o da direita o deslocamento. Detalhe que essa instrução não é suportada em 64-bit.

O segundo tipo de _far_ `call`, suportado em 64-bit, é o que recebe como operando um valor na memória. Mas perceba que temos um _near_ `call` que recebe o mesmo tipo de argumento, não é mesmo?

Por padrão o NASM irá montar as instruções como _near_ e não _far_ mas você pode evitar essa ambiguidade explicitando com _keywords_ do NASM que são bem intuitivas. Veja:

```nasm
call [rbx]       ; Próximo e absoluto
call near [rbx]  ; Próximo e absoluto
call far [rbx]   ; Distante
```

O _near_ espera o endereço do _offset_ na memória, não tem segredo. Mas o _far_ espera o _offset_ seguido do segmento. Em um sistema de 32-bit vamos supor que nosso procedimento está no segmento `0xaaaa` e no _offset_ `0xbbbb1111`. Em memória o valor precisa estar assim em _little-endian_:

```
11 11 bb bb aa aa
```

No NASM essa variável poderia ser _dumpada_ da seguinte forma:

```nasm
bits 32

my_addr: dd 0xbbbb1111   ; Deslocamento
         dw 0xaaaa       ; Segmento

; E usada assim:
call far [my_addr]
```

Basicamente o _far_ `call` modifica o valor de CS e IP ao mesmo tempo, enquanto o _near_ `call` apenas modifica o valor de IP.

{% hint style="info" %}
No código de máquina a diferença entre o _far_ e o _near_ **call** que usam o operando em memória está no campo REG do byte ModR/M. O _near_ tem o valor 2 e o _far_ tem o valor 3. O opcode é **0xFF**.

Se você não entendeu isso aqui, não se preocupa com isso. Mais para frente no livro será escrito um capítulo só para explicar o código de máquina da arquitetura.
{% endhint %}

### RET

```
ret
retf
retn
ret  imm16
retf imm16
retn imm16
```

Como talvez você já tenha reparado intuitivamente a chamada _far_ também preserva o valor de CS na _stack_ e não apenas o valor de IP (lembrando que IP já estaria apontando para a instrução seguinte na memória).

Por isso a instrução `ret` também precisa ser diferente dentro de um procedimento que será chamado com um _far call_. Ao invés de apenas ler o _offset_ na _stack_ ela precisa ler o segmento também, assim modificando CS e IP do mesmo jeito que o `call`.

Repetindo que o NASM por padrão irá montar as instruções como _near _então precisamos especificar para o NASM, em um procedimento que deve ser chamado como _far_,_ _que queremos usar um `ret` _far_.\
Para isso podemos simplesmente adicionar um sufixo 'n' para especificar como _near_, que já é o padrão, ou o sufixo 'f' para especificar como _far_. Ficando:

```nasm
retf  ; Usado em procedimentos que devem ser chamados com far call
```

Existe também uma outra opção de instrução `ret` que recebe como operando um valor imediato de 16-bit que especifica um número de bytes a serem desempilhados da _stack_.

Basicamente o que ele faz é somar o valor de SP com esse número, porque como sabemos a pilha cresce "para baixo". Ou seja se subtraímos valor em SP estamos fazendo a pilha crescer, se somamos estamos fazendo ela diminuir. Por exemplo, podemos escrever em pseudo-código a instrução `retf 12` da seguinte forma:

{% code title="pseudo.c" %}
```c
RIP = pop();
CS  = pop();
RSP = RSP + 12;
```
{% endcode %}
