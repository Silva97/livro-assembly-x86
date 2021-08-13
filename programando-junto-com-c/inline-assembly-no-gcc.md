---
description: Aprendendo a usar o inline Assembly do compilador GCC.
---

# Inline Assembly no GCC

_Inline Assembly_ é uma extensão do compilador que permite inserir código Assembly diretamente no código de saída do compilador. Dessa forma é possível misturar C e Assembly sem a necessidade de usar um módulo separado só para o código em Assembly, além de permitir alguns recursos interessantes que não são possíveis sem o _inline_ Assembly.

{% hint style="info" %}
O compilador Clang contém uma sintaxe de _inline_ Assembly compatível com a do GCC, logo o conteúdo ensinado aqui também é válido para o Clang.
{% endhint %}

## Inline Assembly básico

A sintaxe do uso básico é: `asm [qualificadores] ( instruções-asm )`.

Onde qualificadores é uma \(ou mais\) das seguintes palavra-chaves:

* **volatile**: Isso desabilita as otimizações de código no _inline_ Assembly, mas esse já é o padrão quando se usa o _inline_ ASM básico.
* **inline**: Isso é uma "dica" para o compilador considerar que o tamanho do código Assembly é o menor possível. Serve meramente para o compilador decidir se vai ou não expandir uma função como _inline_, e usando esse qualificador você sugere que o código é pequeno o suficiente para isso.

As instruções Assembly ficam dentro dos parênteses como uma _string_ literal e são despejadas no código de saída sem qualquer alteração por parte do compilador. Geralmente se usa `\n\t` para separar cada instrução pois isso vai ser refletido literalmente na saída de código. O `\n` é para iniciar uma nova linha e o `\t` \(TAB\) é para manter a indentação do código de maneira idêntica ao código gerado pelo compilador.

Exemplo:

```c
#include <stdio.h>

int main(void)
{
  asm(
    "mov $5, %eax\n\t"
    "add $3, %eax\n\t"
  );

  return 0;
}
```

Isso produz a seguinte saída ao [visualizar o código de saída](./#vendo-o-codigo-de-saida-do-gcc):

```text
main:
	endbr64
	pushq	%rbp
	movq	%rsp, %rbp
#APP
# 5 "main.c" 1
	mov $5, %eax
	add $3, %eax
	
# 0 "" 2
#NO_APP
	movl	$0, %eax
	popq	%rbp
	ret
```

Entre as diretivas `#APP` e `#NO_APP` fica o código despejado do _inline_ Assembly. A diretiva `# 5 "main.c" 1` é apenas um atalho para a diretiva `#line` onde ela serve para avisar para o assembler de qual linha \(5\) e arquivo \("main.c"\) veio aquele código. Assim se ocorrer algum erro, na mensagem de erro do assembler será exibido essas informações.

Repare que o _inline_ Assembly apenas despeja literalmente o conteúdo da _string_ literal. Logo você pode adicionar o que quiser aí incluindo diretivas, comentários ou até mesmo instruções inválidas que o compilador não irá reclamar.

Também é possível usar _inline_ Assembly básico fora de uma função, como em:

```c
#include <stdio.h>

int add(int, int);

int main(void)
{
  printf("%d\n", add(2, 3));
  return 0;
}

asm (
  "add:\n\t"
  "  lea (%edi, %esi), %eax\n\t"
  "  ret"
);
```

Porém não é possível fazer o mesmo com _inline_ Assembly estendido.

## Inline Assembly estendido

A versão estendida do _inline_ Assembly funciona de maneira semelhante ao _inline_ Assembly básico, porém com a diferença de que é possível acessar variáveis em C e fazer saltos para rótulos no código fonte em C.

A sintaxe da versão estendida segue o seguinte formato:

```c
asm [qualificadores] (
  "instruções-asm"
  : operandos-de-saída
  : operandos-de-entrada
  : clobbers
  : rótulos-goto
)
```

Os qualificadores são os mesmos da versão básica porém com mais um chamado **goto**. O qualificador **goto** indica que o código Assembly pode efetuar um salto para um dos rótulos listados no último operando. Esse qualificador é necessário para se usar os rótulos no código ASM.

Dentre esses operandos somente os de saída são "obrigatórios", os demais podem ser omitidos. E todos eles podem conter uma lista vazia exceto o de rótulos.

Existe um limite **máximo de 30 operandos** com a soma dos operandos de saída, entrada e rótulos.

### operandos-de-saída

Cada operando de entrada é separado por vírgula e contém a seguinte sintaxe:

```c
[nome] "restrições" (variável)
```

Onde `nome` é um símbolo opcional que você pode criar para se referir ao operando no código Assembly. Também é possível se referir ao operando usando `%n`, onde **n** seria o índice do operando \(contando a partir de zero\). E usar `%[nome]` caso defina um nome.

Como o `%` é usado para se referir à operandos, no _inline_ Assembly estendido se usa dois `%` para se referir à um registrador. Já que `%%` é um escape para escrever o próprio `%` na saída da mesma forma que se faz na função **printf**.

 [As restrições](inline-assembly-no-gcc.md#restricoes) é uma _string_ literal contendo letras e símbolos indicando como esse operando deve ser armazenado \(**r** para registrador e **m** para memória, por exemplo\). No caso dos operandos de entrada o primeiro caractere na _string_ deve ser um `=` ou `+`. Onde o `=` indica que a variável terá seu valor modificado, enquanto `+` indica que terá seu valor modificado e lido.

Operandos de saída com `+` são contabilizados como dois, tendo em vista que o `+` é basicamente um atalho para repetir o mesmo operando também como uma entrada.

Essas informações são necessárias para que o compilador consiga otimizar o código corretamente. Por exemplo caso você indique que a variável será somente escrita com `=` mas leia o valor da variável no Assembly, o compilador pode assumir que o valor da variável nunca foi lido e portanto descartar a inicialização dela durante a otimização de código. Isso criaria um comportamento estranho no _inline_ Assembly onde se obteria lixo como valor da variável.

Um exemplo deste erro:

```c
#include <stdio.h>

int main(void)
{
  int x = 5;

  asm("addl $3, %0"
      : "=rm"(x));

  printf("%d\n", x);
  return 0;
}
```

A otimização de código pode remover a inicialização `x = 5` já que não informamos que o valor dessa variável é lido dentro no _inline_ Assembly. O correto seria usar `+` nesse caso.

Um exemplo \(dessa vez correto\) usando um símbolo definido para o operando:

```c
#include <stdio.h>

int main(void)
{
  int x;

  asm("movl $5, %[myvar]"
      : [myvar] "=rm"(x));

  printf("%d\n", x);
  return 0;
}
```

{% hint style="info" %}
Caso utilize um operando que você não tem **certeza** que será armazenado em um registrador, lembre-se de usar o sufixo na instrução para especificar o tamanho do operando. Para evitar erros é ideal que sempre use os sufixos.
{% endhint %}

### operandos-de-entrada

Os operandos de entrada seguem a mesma sintaxe dos operandos de saída porém sem o `=` ou `+` nas restrições. Não se deve tentar modificar operandos de entrada \(embora tecnicamente seja possível\) para evitar erros, lembre-se que o compilador irá otimizar o código assumindo que aquele operando não será modificado.

Também é possível passar expressões literais como operando de entrada ao invés de somente nomes de variáveis. A expressão será avaliada e seu valor passado como operando sendo armazenado de acordo com as restrições.

### clobbers

O _clobbers_ \(que eu não sei como traduzir\) é basicamente uma lista, separada por vírgula, de efeitos colaterais do código Assembly. Nele você **deve** listar o que o seu código ASM modifica além dos operandos de saída. Cada valor de _clobber_ é uma _string_ literal contendo o nome de um registrador que é modificado pelo seu código. Também há dois nomes especiais de _clobbers_:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Clobber</th>
      <th style="text-align:left">Descri&#xE7;&#xE3;o</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">cc</td>
      <td style="text-align:left">Indica que o c&#xF3;digo ASM modifica <em>flags</em> do processador.</td>
    </tr>
    <tr>
      <td style="text-align:left">memory</td>
      <td style="text-align:left">
        <p>Indica que o c&#xF3;digo ASM faz leitura ou escrita da/na mem&#xF3;ria
          em outro lugar que n&#xE3;o seja um dos operandos de entrada ou sa&#xED;da.
          Por exemplo em uma mem&#xF3;ria apontada por um ponteiro de um operando.</p>
        <p></p>
        <p>Esse <em>clobber</em> evita que o compilador assuma que os valores das vari&#xE1;veis
          na mem&#xF3;ria permanecem o mesmo ap&#xF3;s a execu&#xE7;&#xE3;o do c&#xF3;digo
          ASM. E tamb&#xE9;m garante que o compilador escreva o valor de todas as
          vari&#xE1;veis na mem&#xF3;ria antes de executar o <em>inline </em>ASM.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">rax</td>
      <td style="text-align:left">Indica que o registrador RAX ser&#xE1; modificado.</td>
    </tr>
    <tr>
      <td style="text-align:left">rbx</td>
      <td style="text-align:left">Indica que o registrador RBX ser&#xE1; modificado.</td>
    </tr>
    <tr>
      <td style="text-align:left">etc.</td>
      <td style="text-align:left">...</td>
    </tr>
  </tbody>
</table>

Qualquer nome de registrador é válido para ser usado como _clobber_ exceto o _Stack Pointer_ \(RSP\). É esperado que no final da execução do _inline_ ASM o valor de RSP seja o mesmo de antes da execução do código. Se não for o compilador pode gerar código que criará problemas na execução.

Quando você adiciona um registrador a lista de _clobbers_ ele não será utilizado para armazenar operandos de entrada ou saída, assim garantindo que o registrador pode ser utilizado livremente no _inline_ ASM sem causar qualquer erro.

Exemplo:

```c
int add(int a, int b)
{
  int result;

  asm("movl %[a], %%eax\n\t"
      "addl %[b], %%eax\n\t"
      "movl %%eax, %[result]"
      : [result] "=rm"(result)
      : [a] "r"(a),
        [b] "r"(b)
      : "cc",
        "eax");

  return result;
}
```

### rótulos-goto

Ao usar `asm goto` pode-se referir à um rótulo usando o prefixo `%l` seguido do índice do operando de rótulo. Onde a contagem inicia em zero e é contabilizado também os operandos de entrada e saída.

Exemplo:

```c
#include <stdio.h>
#include <stdbool.h>

int do_anything(bool value)
{
  int result = 3;

  asm goto("test %[value], %[value]\n\t"
           "jz %l1"
           :
           : [value] "r"(value)
           : "cc"
           : my_label);

  result += 2;

my_label:
  return result;
}

int main(void)
{
  printf("%d, %d\n", do_anything(true), do_anything(false));
  return 0;
}
```

Mas felizmente também é possível usar o nome do rótulo no _inline_ Assembly, bastando usar a notação `%l[nome]`. O exemplo acima poderia ter a instrução de salto reescrita para `jz %l[my_label]`.

## Restrições

As restrições \(_constraints_\) são uma lista de caracteres que determinam onde um operando deve ser armazenado. É possível indicar múltiplas alternativas para o compilador simplesmente adicionando mais de uma letra indicando tipos de armazenamento diferentes.

Abaixo a lista de algumas restrições disponíveis no GCC.

### Restrições comuns

| Restrição | Descrição |
| :--- | :--- |
| `m` | Operando na memória. |
| `r` | Operando em um registrador de propósito geral. |
| `i` | Um valor inteiro imediato. |
| `F` | Um valor _floating-point_ imediato. |
| `g` | Um operando na memória, registrador de propósito geral ou inteiro imediato. Mesmo efeito que usar `"rim"` como restrição. |
| `p` | Um operando que é um endereço de memória válido. |
| `X` | Qualquer operando é permitido. Basicamente deixa a decisão nas mãos do compilador. |

### Restrições para família x86

| Restrição | Descrição |
| :--- | :--- |
| `R` | Registradores legado. Qualquer um dos oito registradores disponíveis em IA-32. |
| `q` | Qualquer registrador que seja possível ler o byte menos significativo. Como RAX \(AL\) ou R8 \(R8B\) por exemplo. |
| `Q` | Qualquer registrador que seja possível ler o segundo byte menos significativo, como RAX \(AH\) por exemplo. |
| `a` | O registrador "A" \(RAX, EAX, AX ou AL\). |
| `b` | O registrador "B" \(RBX, EBX, BX ou BL\). |
| `c` | O registrador "C" \(RCX, ECX, CX ou CL\). |
| `d` | O registrador "D" \(RDX, EDX, DX ou DL\). |
| `S` | RSI, ESI, SI ou SIL. |
| `D` | RDI, EDI, DI ou DIL. |
| `A` | O conjunto AX:DX. |
| `f` | Qualquer [registrador do x87](../aprofundando-em-assembly/usando-instrucoes-da-fpu.md#registradores). |
| `t` | ST0 |
| `u` | ST1 |
| `y` | Qualquer registrador MMX. |
| `x` | Qualquer [registrador SSE](../aprofundando-em-assembly/entendendo-sse/#registradores-xmm). |
| `Yz` | XMM0 |
| `I` | Um inteiro constante entre 0 e 31, usado para _shift_ com valores de 32-bit. |
| `J` | Um inteiro constante entre 0 e 63, usado para _shift_ com valores de 64-bit. |
| `K` | Inteiro sinalizado de 8-bit. |
| `N` | Inteiro não-sinalizado de 8-bit. |

## Dicas

### Rótulos locais no inline Assembly

Se você simplesmente declarar rótulos dentro do _inline_ Assembly pode acabar se deparando com uma redeclaração de símbolo por não ter uma garantia de que ele seja único. Mas uma dica é usar o escape especial `%=` que expande para um número único para cada uso de `asm`, assim sendo possível dar um nome único para os rótulos.

Exemplo:

```c
#include <stdio.h>
#include <stdbool.h>

int do_anything(bool value)
{
  int result = 3;

  asm("test %[value], %[value]\n\t"
      "jz .my_label%=\n\t"
      "addl $2, %[result]\n\t"
      ".my_label%=:"
      : [result] "+a"(result)
      : [value] "r"(value)
      : "cc");

  return result;
}

int main(void)
{
  printf("%d, %d\n", do_anything(true), do_anything(false));
  return 0;
}
```

### Usando sintaxe Intel

Caso prefira usar sintaxe Intel é possível fazer isso meramente compilando o código com `-masm=intel`. Isso porque o _inline_ Assembly simplesmente despeja as instruções no arquivo de saída, portanto o código irá usar a sintaxe que o _assembler_ utilizar.

Outra dica é usar a diretiva `.intel_syntax noprefix` no início e depois `.att_syntax` no final para religar a sintaxe AT&T para o restante do código. Exemplo:

```c
int add(int a, int b)
{
  int result;

  asm(".intel_syntax noprefix\n\t"
      "lea %[result], [ %[a] + %[b] ]\n\t"
      ".att_syntax"
      : [result] "=a"(result)
      : [a] "r"(a),
        [b] "r"(b));

  return result;
}
```

### Escolhendo o registrador/símbolo para uma variável

Ao usar o _storage-class_ `register` é possível escolher em qual registrador a variável será armazenada usando a seguinte sintaxe:

```c
register int x asm("r12") = 5;
```

Nesse exemplo a variável `x` **obrigatoriamente** seria alocada no registrador R12.

Também é possível escolher o nome do símbolo para variáveis locais com _storage-class_ `static` ou para variáveis globais. Como em:

```c
static int x asm("my_var") = 5;
```

A variável no código fonte é referida como `x` mas o símbolo gerado para a variável seria definido como `my_var`.

