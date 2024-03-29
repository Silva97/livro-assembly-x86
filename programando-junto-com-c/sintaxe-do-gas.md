---
description: Aprendendo a sintaxe AT&T e a usar o GAS
---

# Sintaxe do GAS

O GNU assembler (GAS) usa por padrão a sintaxe AT\&T e neste tópico irei ensiná-la. Mais abaixo irei ensinar a diretiva usada para usar sintaxe Intel meramente como curiosidade e caso prefira usá-la.

## Diferenças entre sintaxe Intel e AT\&T

A primeira diferença notável é que o operando destino nas instruções de sintaxe Intel é o mais à esquerda, o primeiro operando. Já na sintaxe da AT\&T é o inverso, o operando mais à direita é o operando destino. Conforme exemplo:

```
# Isso é um comentário.

# Sintaxe AT&T       # Sintaxe Intel

mov $123, %eax       # mov eax, 123
mov $0x0A, %eax      # mov eax, 0x0A
mov $0b1010, %eax    # mov eax, 0b1010
mov $'A', %eax       # mov eax, 'A'
```

E como já pode ser observado valores literais precisam de um prefixo `$`, enquanto os nomes dos registradores precisam do prefixo `%`.

### Tamanho dos operandos

Na sintaxe da Intel o tamanho dos operandos é especificado com base em palavra-chaves que são adicionadas anteriormente ao operando. Na sintaxe AT\&T o tamanho do operando é especificado por um sufixo adicionado a instrução, conforme tabela abaixo:

| Sufixo | Tamanho                   | Palavra-chave equivalente no NASM |
| ------ | ------------------------- | --------------------------------- |
| B      | byte (8 bits)             | byte                              |
| W      | word (16 bits)            | word                              |
| L      | long/doubleword (32 bits) | dword                             |
| Q      | quadword (64 bits)        | qword                             |
| T      | ten word (80 bits)        | tword                             |

Exemplos:

```
# AT&T               # Intel

movl $5, %eax        # mov dword eax, 5
movb $'A', (%ebx)    # mov byte [ebx], 'A'
```

Assim como o NASM consegue identificar o tamanho do operando quando é usado um registrador e a palavra-chave se torna opcional, o mesmo acontece no GAS e o sufixo também é opcional nesses casos.

### Far jump e call

Na sintaxe Intel saltos e chamadas distantes são feitas com `jmp far [etc]` e `call far [etc]` respectivamente. Na sintaxe da AT\&T se usa o prefixo **L** nessas instruções, ficando: `ljmp` e `lcall`.

### Endereçamento

Na sintaxe Intel [o endereçamento](../a-base/enderecamento.md) é bem intuitivo já que ele é escrito em formato de expressão matemática. Na sintaxe AT\&T é um pouco mais confuso e segue o seguinte formato:\
`segment:displacement(base, index, scale)`.

Exemplos com o seu equivalente na sintaxe da Intel:

```
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

### Saltos

Na sintaxe da AT\&T os saltos para endereços armazenados na memória devem ter um `*` antes do rótulo para indicar que o salto deve ocorrer para o endereço que está armazenado naquele endereço de memória. Sem o `*` o salto ocorre para o rótulo em si. Exemplo:

```
# AT&T               # Intel

jmp my_code          # jmp my_code
jmp *my_pointer      # jmp [my_pointer]
```

Saltos que especificam segmento e _offset_ separam os dois valores por vírgula. Como em:

```
# AT&T               # Intel

jmp $1234, $5678     # jmp 1234:5678
```

## Aprendendo a usar o GAS

As diretivas do GAS funcionam de maneira semelhante as diretivas do NASM com a diferença que todas elas são prefixadas por um ponto.

### Comentários

No GAS comentários de múltiplas linhas podem ser escritos com `/*` e `*/` assim como em C. Comentários de uma única linha podem ser escritos com `#` ou `//`.

### Pseudo-instruções de dados

No NASM [existem as pseudo-instruções](../a-base/instrucoes-do-nasm.md#variaveis) `db`, `dw`, `dd`, `dq` etc. que servem para despejar bytes no arquivo binário de saída. No GAS isso é feito usando as seguintes pseudo-instruções:

| Pseudo-instrução                           | Tipo do dado (tamanho em bits)                         | Equivalente no NASM |
| ------------------------------------------ | ------------------------------------------------------ | ------------------- |
| .byte                                      | byte (8 bits)                                          | db                  |
| <p>.short</p><p>.hword</p><p>.word</p>     | word (16 bits)                                         | dw                  |
| <p>.long</p><p>.int</p>                    | doubleword (32 bits)                                   | dd                  |
| .quad                                      | quadword (64 bits)                                     | dq                  |
| <p>.float</p><p>.single</p>                | Single-precision floating-point (32 bits)              | dd                  |
| .double                                    | Double-precision floating-point (64 bits)              | dq                  |
| <p>.ascii</p><p>.string</p><p>.string8</p> | String (8 bits cada caractere)                         | db                  |
| .asciz                                     | Mesmo que .ascii porém com um terminador nulo no final | -                   |
| .string16                                  | String (16 bits cada caractere)                        | -                   |
| .string32                                  | String (32 bits cada caractere)                        | -                   |
| .string64                                  | String (64 bits cada caractere)                        | -                   |

Exemplos:

```
msg1: .ascii "Hello World\0"
msg2: .asciz "Hello World"

value1: .byte 1, 2, 3, 4, 5
value2: .float 3.1415
```

### Diretivas de seções e alinhamento

O GAS tem diretivas específicas para declarar algumas seções padrão. Conforme tabela:

| GAS   | Equivalente no NASM |
| ----- | ------------------- |
| .data | section .data       |
| .bss  | section .bss        |
| .text | section .text       |

Porém ele também tem a diretiva `.section` que pode ser usada de maneira semelhante a `section` do NASM. Os atributos da seção porém são passados em formato de _flags_ em uma string como segundo argumento. As _flags_ principais são `w` para dar permissão de escrita e `x` para dar permissão de execução. Exemplos:

```
# GAS                         # NASM

.section .mysection, "wx"     # section .mysection write exec
.section .rodata              # section .rodata
.text                         # section .text
.section .text                # section .text
```

A diretiva `.align` pode ser usada para alinhamento dos dados. Você pode usá-la no início da seção para alinhar a mesma, conforme exemplo:

```
    .section .rodata
    .align 16

# Equivalente no NASM: section .rodata align=16
```

### Usando sintaxe Intel

A diretiva `.intel_syntax` pode ser usada para habilitar a sintaxe da Intel para o GAS. Opcionalmente pode-se passar um parâmetro `noprefix` para desabilitar o prefixo `%` dos registradores.

Uma diferença importante da sintaxe Intel do GAS em relação ao NASM é que as palavra-chaves que especificam o tamanho do operando precisam ser seguidas por `ptr`, conforme exemplo abaixo:

```
    .intel_syntax noprefix
    .text
example:
    mov byte ptr [ebx], 7
    mov word ptr [ebx], 7
    mov dword ptr [ebx], 7
    mov qword ptr [ebx], 7
    ret
```

## Exemplo de código na sintaxe AT\&T

O exemplo abaixo é o mesmo apresentado no tópico sobre [instruções de movimentação SSE](../aprofundando-em-assembly/entendendo-sse/instrucoes-de-movimentacao-de-dados.md) porém reescrito na sintaxe do GAS/AT\&T:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

void assembly(float *array);

int main(void)
{
  float array[4];
  assembly(array);

  printf("%f, %f, %f, %f\n", array[0], array[1], array[2], array[3]);
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
