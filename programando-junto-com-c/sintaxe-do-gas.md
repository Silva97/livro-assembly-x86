---
description: Aprendendo a sintaxe AT&T e a usar o GAS
---

# Sintaxe do GAS

O GNU assembler \(GAS\) usa por padrão a sintaxe AT&T e neste tópico irei ensiná-la. Mais abaixo irei ensinar a diretiva usada para usar sintaxe Intel meramente como curiosidade e caso prefira usá-la.

## Diferenças entre sintaxe Intel e AT&T

A primeira diferença notável é que o operando destino nas instruções de sintaxe Intel é o mais à esquerda, o primeiro operando. Já na sintaxe da AT&T é o inverso, o operando mais à direita é o operando destino. Conforme exemplo:

```text
# Isso é um comentário.

# Sintaxe AT&T       # Sintaxe Intel

mov $123, %eax       # mov eax, 123
mov $0x0A, %eax      # mov eax, 0x0A
mov $0b1010, %eax    # mov eax, 0b1010
mov $'A', %eax       # mov eax, 'A'
```

E como já pode ser observado valores literais precisam de um prefixo `$`, enquanto os nomes dos registradores precisam do prefixo `%`.

### Tamanho dos operandos

Na sintaxe da Intel o tamanho dos operandos é especificado com base em palavra-chaves que são adicionadas anteriormente ao operando. Na sintaxe AT&T o tamanho do operando é especificado por um sufixo adicionado a instrução, conforme tabela abaixo:

| Sufixo | Tamanho | Palavra-chave equivalente no NASM |
| :--- | :--- | :--- |
| B | byte \(8 bits\) | byte |
| W | word \(16 bits\) | word |
| L | long/doubleword \(32 bits\) | dword |
| Q | quadword \(64 bits\) | qword |

Assim como o NASM muitas vezes consegue identificar o tamanho do operando e a palavra-chave se torna opcional, o mesmo acontece no GAS e o sufixo também é opcional nesses casos.

### Far jump e call

Na sintaxe Intel saltos e chamadas distantes são feitas com `jmp far [etc]` e `call far [etc]` respectivamente. Na sintaxe da AT&T se usa o prefixo **L** nessas instruções, ficando: `ljmp` e `lcall`.

### Endereçamento

Na sintaxe Intel [o endereçamento](../a-base/enderecamento.md) é bem intuitivo já que ele é escrito em formato de expressão matemática. Na sintaxe AT&T é um pouco mais confuso e segue o seguinte formato:  
`segment:displacement(base, index, scale)`.

Exemplos com o seu equivalente na sintaxe da Intel:

```text
# AT&T                       # Intel

mov my_var, %eax             # mov eax, [my_var]
mov 5(%ebx), %eax            # mov eax, [ebx + 5]
mov 5(%ebx, %esi), %eax      # mov eax, [ebx + esi + 5]
mov 5(%ebx, %esi, 4), %eax   # mov eax, [ebx + esi*4 + 5]

mov (, %esi, 4), %eax        # mov eax, [esi*4]
mov (%ebx), %eax             # mov eax, [ebx]

mov %es:(%ebx), %eax         # mov eax, [es:ebx]
mov %es:5(%ebx), %eax        # mov eax, [es:ebx + 5]

mov my_var(%rip), %rax       # mov rax, [rel my_var]
```

Como demonstrado no último exemplo o **endereço relativo** na sintaxe do GAS é feito explicitando RIP como base, enquanto na sintaxe do NASM isso é feito usando a palavra-chave `rel`.

## Aprendendo a usar o GAS

As diretivas do GAS funcionam de maneira semelhante as diretivas do NASM com a diferença que todas elas são prefixadas por um ponto.

### Comentários

No GAS comentários de múltiplas linhas podem ser escritos com `/*` e `*/` assim como em C. Comentários de uma única linha podem ser escritos com `#` ou `//`.

### Pseudo-instruções de dados

No NASM [existem as pseudo-instruções](../a-base/instrucoes-do-nasm.md#variaveis) `db`, `dw`, `dd`, `dq` etc. que servem para despejar bytes no arquivo binário de saída. No GAS isso é feito usando as seguintes pseudo-instruções:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Pseudo-instru&#xE7;&#xE3;o</th>
      <th style="text-align:left">Tipo do dado (tamanho em bits)</th>
      <th style="text-align:left">Equivalente no NASM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">.byte</td>
      <td style="text-align:left">byte (8 bits)</td>
      <td style="text-align:left">db</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>.short</p>
        <p>.hword</p>
        <p>.word</p>
      </td>
      <td style="text-align:left">word (16 bits)</td>
      <td style="text-align:left">dw</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>.long</p>
        <p>.int</p>
      </td>
      <td style="text-align:left">doubleword (32 bits)</td>
      <td style="text-align:left">dd</td>
    </tr>
    <tr>
      <td style="text-align:left">.quad</td>
      <td style="text-align:left">quadword (64 bits)</td>
      <td style="text-align:left">dq</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>.float</p>
        <p>.single</p>
      </td>
      <td style="text-align:left">Single-precision floating-point (32 bits)</td>
      <td style="text-align:left">dd</td>
    </tr>
    <tr>
      <td style="text-align:left">.double</td>
      <td style="text-align:left">Double-precision floating-point (64 bits)</td>
      <td style="text-align:left">dq</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>.ascii</p>
        <p>.string</p>
        <p>.string8</p>
      </td>
      <td style="text-align:left">String (8 bits cada caractere)</td>
      <td style="text-align:left">db</td>
    </tr>
    <tr>
      <td style="text-align:left">.asciz</td>
      <td style="text-align:left">Mesmo que .ascii por&#xE9;m com um terminador nulo no final</td>
      <td style="text-align:left">-</td>
    </tr>
    <tr>
      <td style="text-align:left">.string16</td>
      <td style="text-align:left">String (16 bits cada caractere)</td>
      <td style="text-align:left">-</td>
    </tr>
    <tr>
      <td style="text-align:left">.string32</td>
      <td style="text-align:left">String (32 bits cada caractere)</td>
      <td style="text-align:left">-</td>
    </tr>
    <tr>
      <td style="text-align:left">.string64</td>
      <td style="text-align:left">String (64 bits cada caractere)</td>
      <td style="text-align:left">-</td>
    </tr>
  </tbody>
</table>

Exemplos:

```text
msg1: .ascii "Hello World\0"
msg2: .asciz "Hello World"

value1: .byte 1, 2, 3, 4, 5
value2: .float 3.1415
```

### Diretivas de seções e alinhamento

O GAS tem diretivas específicas para declarar algumas seções padrão. Conforme tabela:

| GAS | Equivalente no NASM |
| :--- | :--- |
| .data | section .data |
| .bss | section .bss |
| .text | section .text |

Porém ele também tem a diretiva `.section` que pode ser usada de maneira semelhante a `section` do NASM. Os atributos da seção porém são passados em formato de _flags_ em uma string como segundo argumento. As _flags_ principais são `w` para dar permissão de escrita e `x` para dar permissão de execução. Exemplos:

```text
# GAS                         # NASM

.section .mysection, "wx"     # section .mysection write exec
.section .rodata              # section .rodata
.text                         # section .text
.section .text                # section .text
```

A diretiva `.align` pode ser usada para alinhamento dos dados. Você pode usá-la no início da seção para alinhar a mesma, conforme exemplo:

```text
    .section .rodata
    .align 16

# Equivalente no NASM: section .rodata align=16
```

### Usando sintaxe Intel

A diretiva `.intel_syntax` pode ser usada para habilitar a sintaxe da Intel para o GAS. Opcionalmente pode-se passar um parâmetro `noprefix` para desabilitar o prefixo `%` dos registradores.

Uma diferença importante da sintaxe Intel do GAS em relação ao NASM é que as palavra-chaves de especificam o tamanho do operando precisa ser seguidos por `ptr`, conforme exemplo abaixo:

```text
    .intel_syntax noprefix
    .text
example:
    mov byte ptr [ebx], 7
    mov word ptr [ebx], 7
    mov dword ptr [ebx], 7
    mov qword ptr [ebx], 7
    ret
```

## Exemplo de código na sintaxe AT&T

O exemplo abaixo é o mesmo apresentado no tópico sobre [instruções de movimentação SSE](../aprofundando-em-assembly/entendendo-sse/instrucoes-de-movimentacao-de-dados.md) porém reescrito na sintaxe do GAS/AT&T:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

void assembly(float *array);

int main(void)
{
  float array[4];
  assembly(array);

  printf("Resultado: %f, %f, %f, %f\n", array[0], array[1], array[2], array[3]);
  return 0;
}

```
{% endtab %}

{% tab title="assembly.s" %}
```
    .section .rodata
    .align 16
    local_array: .float 1.23
                 .float 2.45
                 .float 3.67
                 .float 4.89

    .text
    .globl assembly
assembly:
    movaps local_array(%rip), %xmm5
    movaps %xmm5, (%rdi)
    ret
```
{% endtab %}
{% endtabs %}

