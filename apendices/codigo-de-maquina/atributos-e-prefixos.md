---
description: Entendendo os prefixos no código de máquina.
---

# Atributos e prefixos

{% hint style="info" %}
Os dois tópicos [atributos](../../aprofundando-em-assembly/atributos.md) e [prefixos](../../aprofundando-em-assembly/prefixos.md) já explicaram esse assunto antes no livro, mas do ponto de vista do Assembly. Aqui será abordado o assunto mais voltado ao código de máquina e **com mais informações**.
{% endhint %}

Na arquitetura x86 as instruções contém o que é conhecido como "atributos", onde existe um determinado valor padrão para o atributo e é possível modificá-lo com um prefixo.

Como pode ser observado na ilustração exibida no tópico [Formato das instruções](formato-das-instrucoes.md), prefixos são bytes que podem (são opcionais na grande maioria das instruções) ser adicionados antes do _opcode_ de uma instrução.

Uma instrução pode ter mais de um prefixo (até 4 legados). O prefixo REX existente somente em x86-64 precisa obrigatoriamente vir antes do _opcode_ e depois dos demais prefixos. Mas exceto por ele, todos os outros prefixos podem ser adicionados em qualquer ordem que não fará diferença na instrução. Por exemplo a instrução `mov eax, [ebx]` em modo de 16-bit seria compilada como na imagem:

![Print do x86-visualizer.](../../.gitbook/assets/Captura\_de\_tela\_de\_2022-04-03\_15-02-52.png)

Onde `67 66 8B 03` e `66 67 8B 03` dariam na mesma, o processador executaria as duas instruções de maneira totalmente equivalente.

### Atributo address-size

{% content-ref url="../../a-base/modos-de-operacao.md" %}
[modos-de-operacao.md](../../a-base/modos-de-operacao.md)
{% endcontent-ref %}

Em modo de 16-bit e modo de 32-bit, desde o processador i386, é possível usar tanto [endereçamento](../../a-base/enderecamento.md) de 16-bit como de 32-bit. No exemplo anterior a instrução `mov eax, [ebx]` foi compilada no modo de 16-bit, porém usando endereçamento e operando de 32-bit.

O atributo **address-size** determina o modo de endereçamento da instrução. Em modo 16-bit o atributo **address-size** por padrão é de 16-bit. E em modo de 32-bit o atributo é por padrão de 32-bit. Já em modo de 64-bit o endereçamento padrão é 64-bit.

O prefixo conhecido como **address-size override**, cujo o byte é `67`, serve para usar o modo de endereçamento não-padrão. Ou seja, ao usar o prefixo se estiver em modo de 16-bit o endereçamento será de 32-bit. E se estiver em modo de 32-bit o endereçamento será de 16-bit. Já se estiver em modo de 64-bit o endereçamento será de 32-bit.

Por isso o prefixo é adicionado em 16-bit para instruções que usam endereçamento de 32-bit. O mesmo também é feito na situação oposta:

![Print do x86-visualizer.](../../.gitbook/assets/Captura\_de\_tela\_de\_2022-04-03\_15-12-37.png)

### Atributo operand-size

Assim como é possível alternar entre endereçamento de 16-bit e 32-bit nos modos de 16-bit (_real mode_) e 32-bit (_protected mode_). Também é possível alternar o tamanho dos operandos usados em operações.

Assim como também foi demonstrado no primeiro exemplo a instrução de 16-bit fez uma operação com um valor de 32-bit (o registrador EAX teve seu valor alterado para os 4 bytes presentes no endereço `[EBX]`).

E para isso foi usado o prefixo **operand-size override**, o byte `66`. E na mesma lógica do `address-size override` ele alterna o tamanho do operando para o seu tamanho não-padrão. Onde em modos de 32-bit e 64-bit o tamanho padrão de operando é de 32-bit, e em modo de 16-bit o tamanho padrão é de 16-bit.

{% hint style="info" %}
Vale citar um erro que eu vi um senhor cometer uma vez: Ele acreditava que em modo de 32-bit era possível usar registradores de 64-bit e endereçamento de 64-bit. Bem, isso está **errado** como você pode notar pela explicação acima.\
\
Em modo de 16-bit é possível usar registradores e endereçamento de 32-bit alterando os atributos **address-size** e **operand-size**. Mas o mesmo não se aplica para 64-bit porque o uso de operandos de 64-bit é feito por meio do prefixo REX, que **só existe em modo de 64-bit**. E em modo de 32-bit só é possível alternar entre endereçamento de 32-bit e 16-bit usando o prefixo `67`.
{% endhint %}

### Atributo segment

{% content-ref url="../../aprofundando-em-assembly/registradores-de-segmento.md" %}
[registradores-de-segmento.md](../../aprofundando-em-assembly/registradores-de-segmento.md)
{% endcontent-ref %}

Qual segmento de memória será acessado pela instrução é definido em um atributo. O segmento padrão da instrução é definido de acordo com qual registrador foi usado como **base**:

| Registrador base           | Segmento |
| -------------------------- | -------- |
| RIP                        | CS       |
| SP/ESP/RSP                 | SS       |
| BP/EBP/RBP                 | SS       |
| Qualquer outro registrador | DS       |

Para alterar o atributo de segmento para um outro segmento de memória é usado um prefixo distinto por segmento:&#x20;

| Segmento | Byte do prefixo |
| -------- | --------------- |
| CS       | `2E`            |
| DS       | `3E`            |
| ES       | `26`            |
| FS       | `64`            |
| GS       | `65`            |
| SS       | `36`            |

Exemplo:

![Print do x86-visualizer.](../../.gitbook/assets/Captura\_de\_tela\_de\_2022-04-03\_15-39-33.png)

### Prefixos REP/REPE e REPNE

As instruções de movimentação de dados (`movsb`, `movsw`, `movsd` e `movsq`) bem como outras como `scasb`, `lodsb`, `in`, `out` etc. podem ser executadas em _loop_ usando o prefixo REPE ou REPNE.

No caso das instruções `MOVS*` é possível usar o prefixo REPE, que nesse caso também pode ser chamado só de `REP` mas os dois mnemônicos produzem o mesmo byte (`F3`).

Ao usar esse prefixo na instrução, assim como foi [explicado anteriormente](formato-das-instrucoes.md#cisc), ela é executada em _loop_ enquanto o valor de ECX não for zero. E a cada iteração do _loop_ o valor do registrador é decrementado. Na verdade se CX ou ECX será usado isso é definido pelo atributo **address-size** e pode ser alternado com o prefixo **address-size override**. Por exemplo na sintaxe do NASM ficaria assim:

```nasm
bits 16
; ...
a32 rep movsb
```

Assim ECX seria usado ao invés de CX. Onde `a32` é uma palavra-chave usada no NASM para denotar que o **address-size** daquela instrução deve ser de 32-bit. Se usado em modo de 16-bit ele adiciona o prefixo `67`, mas se estiver em modo de 32-bit então nenhum prefixo será adicionado tendo em vista que o **address-size** padrão já é de 32-bit.

{% hint style="info" %}
Sim, também existe `a16` e `a64`. Como também existe `o16`, `o32` e `o64` para denotar o tamanho do **operand-size**. Mas detalhe que `a64` e `o64` denotam o uso do prefixo REX que só existe em modo de 64-bit.
{% endhint %}

Nas instruções `CMPS*` e `SCAS*` o prefixo `REPE` (ou `REPZ`) repete a instrução enquanto a [_zero flag_](../../aprofundando-em-assembly/flags-do-processador.md#status-flags) estiver setada. Já `REPNE` (ou `REPNZ`) repete enquanto a _zero flag_ estiver zerada.

### Prefixo LOCK

O prefixo LOCK (byte `F0`) é usado para fazer operações de escrita atômica em um determinado endereço de memória. Ou seja o prefixo garante que outros núcleos do processador não escrevam naquele endereço ao mesmo tempo, exigindo que essa operação finalize antes de outra que escreva no mesmo endereço seja executada.

Esse prefixo só pode ser usado nas seguintes instruções: `ADD`, `ADC`, `AND`, `BTC`, `BTR`, `BTS`, `CMPXCHG`, `CMPXCH8B`, `CMPXCHG16B`, `DEC`, `INC`, `NEG`, `NOT`, `OR`, `SBB`, `SUB`, `XOR`, `XADD` e `XCHG`. Isso, obviamente, quando o operando destino (o que está sendo escrito) é um operando na memória.

Na sintaxe do NASM o prefixo pode ser usado simplesmente com a palavra-chave `lock` antes da instrução. Como em:

```nasm
lock add [ebx], 4
```

### Prefixos de branch hint

É possível manualmente você instruir para o sistema de _**branch prediction**_ do processador quais saltos condicionais provavelmente irão ocorrer ou não usando dois prefixos:

* `2E` - Instrui para o processador que o pulo provavelmente **não** ocorrerá.
* `3E` - Instrui para o processador que provavelmente o pulo ocorrerá.

Na sintaxe do NASM esses prefixos podem ser adicionados em saltos condicionais com as palavra-chaves `false` e `true` respectivamente. Como em:

```
false jz my_label
```

Todavia esses prefixos são obsoletos e até mesmo ignorados por processadores mais novos, tendo em vista que processadores mais modernos usam um algoritmo para determinar qual salto é mais provável de ser tomado ou não. E também saltos para trás são considerados tomados e saltos para frente como não tomados. Isso por causa da forma como compiladores geram código para _loops_ e condicionais.

Em versões mais modernas do NASM ele simplesmente irá ignorar o `false` ou `true` e não adicionará prefixo algum.
