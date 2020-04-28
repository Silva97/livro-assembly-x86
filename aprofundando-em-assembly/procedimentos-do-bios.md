---
description: >-
  Existem algumas interrupções que são criadas pelo próprio BIOS do sistema.
  Vamos ver algumas delas aqui.
---

# Procedimentos do BIOS

BIOS — _Basic Input/Output System_ — é o firmware da placa-mãe responsável pela inicialização do hardware. Ele quem começa o processo de _boot_ do sistema, além de anteriormente fazer um teste rápido \(POST — _Power-On Self Test_\) para verificar se o hardware está funcionando apropriadamente.

Mas além de fazer essa tarefa de inicialização do PC, ele também define algumas interrupções que podem ser usadas pelo software em 16-bit para tarefas básicas. E é daí que vem seu nome, já que estas tarefas são operações básicas de entrada e saída de dados para o hardware.

Cada interrupção não faz um procedimento único, mas sim vários procedimentos relacionados à um determinado hardware. Qual procedimento especificamente será executado é, na maioria das vezes, definido no registrador `AH`ou `AX`.

### INT 0x10

Esta interrupção tem procedimentos relacionados ao vídeo, como a escrita de caracteres na tela ou até mesmo alterar o modo de vídeo.

#### AH 0x0E

O procedimento INT 0x10 / AH 0x0E simplesmente escreve um caractere na tela em modo _teletype_, que é um nome chique para dizer que o caractere é impresso na posição atual do cursor e atualiza a posição do mesmo. É algo bem semelhante ao que a gente vê sobre um sistema operacional usando uma função como `putchar()`em C.

Esse procedimento recebe como argumento no registrador `AL`o caractere a ser impresso, e em `BH`o número da página.

O número da página varia entre 0 e 7. São 8 páginas diferentes que podem ser apresentadas para o monitor como o conteúdo da tela. Por padrão é usada a página 0, mas você pode alternar entre as páginas fazendo com que conteúdo diferente seja apresentado na tela sem perder o conteúdo da outra página.

Se você já usou o MS-DOS deve ter visto programas, como editores de código, que imprimiam uma interface de texto \(TUI\), mas depois que finalizava o conteúdo do prompt voltava para a tela... Esses programas basicamente alternavam de página.

{% code title="exemplo.asm" %}
```text
mov ah, 0x0E
mov al, 'H'
int 0x10

mov al, 'i'
int 0x10

ret
```
{% endcode %}

No exemplo acima usamos a interrupção duas vezes para imprimir dois caracteres diferentes, fazendo assim um "Hello World" de míseros 11 bytes.

Poderíamos fazer um procedimento para escrever uma _string_ inteira usando um _loop_. Ficaria assim:

{% code title="hello.asm" %}
```text
bits 16
org  0x100

mov si, string
call echo

ret

string: db "Hello World!", 0

; SI = ASCIIZ string
; BH = Página
echo:
  mov ah, 0x0E

.loop:
  lodsb
  test al, al
  jz .stop
  
  int 0x10
  jmp .loop
  
  
.stop:
  ret
```
{% endcode %}

#### AH 0x02

| AH | BH | DH | DL |
| :--- | :--- | :--- | :--- |
| 0x02 | Página | Linha | Coluna |

Esse procedimento seta a posição do cursor em uma determinada página.

#### AH 0x03

| AH | BH |
| :--- | :--- |
| 0x03 | Página |

Pega a posição atual do cursor na página especificada. Retornando:

| CH | CL | DH | DL |
| :--- | :--- | :--- | :--- |
| _Scanline_ inicial | _Scanline_ final | Linha | Coluna |

#### AH 0x05

| AH | AL |
| :--- | :--- |
| 0x05 | Página |

Alterna para a página especificada por AL, que deve ser um número entre 0 e 7.

#### AH 0x09

| AH | AL | BH | BL | CX |
| :--- | :--- | :--- | :--- | :--- |
| 0x09 | Caractere | Página | Atributo | Vezes para imprimir o caractere |

Imprime o caractere AL na posição atual do cursor CX vezes, sem atualizar o cursor. BL é o atributo do caractere, que será explicado mais embaixo.

#### AH 0x0A

| AH | AL | BH | CX |
| :--- | :--- | :--- | :--- |
| 0x0A | Caractere | Página | Vezes para imprimir |

Mesma coisa que o procedimento anterior, mudando somente que não é especificado um atributo para o caractere.

#### AH 0x13

| Registrador | Parâmetro |
| :--- | :--- |
| AL | Modo de escrita |
| BH | Página |
| BL | Atributo |
| CX | Tamanho da _string_ |
| DH | Linha |
| DL | Coluna |
| ES:BP | Endereço da _string_ |

Este procedimento imprime uma _string_ na tela podendo ser especificado um atributo. O modo de escrita pode variar entre 0 e 3, se trata de 2 bits especificando duas informações diferentes:

| Bit | Informação |
| :--- | :--- |
| 0 | Se ligado, atualiza a posição do cursor. |
| 1 | Se desligado, BL é usado para definir o atributo. Se ligado, o atributo é lido da _string_. |

No caso do segundo bit, se estiver ligado então o procedimento irá ler a _string_ considerando que se trata de uma sequência de caractere e atributo. Assim cada caractere pode ter um atributo diferente. Conforme exemplo abaixo:

```c
str: db 'A', 0x05, 'B', 0x0C, 'C', 0x0A
```

#### Caracteres de Ação

Os procedimentos 0x0E e 0x13 interpretam caracteres especiais como determinadas ações que devem ser executadas ao invés de imprimir o caractere na tela. Cada caractere faz uma ação diferente, conforme tabela abaixo:

| Caractere | Nome | Seq. de escape | Ação |
| :--- | :--- | :--- | :--- |
| 0x07 | Bell | \a | Emite um beep. |
| 0x08 | Backspace | \b | Retorna o cursor uma posição. |
| 0x09 | Horizontal TAB | \t | Avança o cursor 4 posições. |
| 0x0A | Line feed | \n | Move o cursor verticalmente para a próxima linha. |
| 0x0D | Carriage return | \r | Move o cursor para o início da linha. |

{% hint style="info" %}
Você pode combinar 0x0D e 0x0A para fazer uma quebra de linha.
{% endhint %}

### INT 0x16

Os procedimentos definidos nesta interrupção são todos relacionados à entrada do teclado. Toda vez que o usuário pressiona uma tecla, esta é lida e armazenada no _buffer_ do teclado. Se você tentar ler do _buffer_ sem haver dados lá, então o sistema irá ficar esperando o usuário inserir uma entrada.

#### AH 0x00

Lê um caractere do _buffer_ do teclado e o remove de lá. Retorna os seguintes valores:

| Registrador | Valor |
| :--- | :--- |
| AL | Código ASCII do caractere |
| AH | _Scancode_ da tecla. \* |

_Scancode_ é um número que identifica a tecla e não especificamente o caractere inserido.

#### AH 0x01

Verifica se há um caractere disponível no _buffer_ sem removê-lo do mesmo. Se houver caractere disponível, retorna:

| Registrador | Valor |
| :--- | :--- |
| AL | Código ASCII |
| AH | _Scancode_ |

O procedimento também modifica a _Zero Flag_ para especificar se há ou não caractere disponível. A define para 0 se houver, caso contrário para 1.

{% hint style="info" %}
Você pode usar em seguida o AH 0x00 para remover o caractere do _buffer_, se assim desejar. Desse jeito é possível pegar um caractere sem fazer uma pausa.
{% endhint %}

#### AH 0x02

Pega _status_ relacionados ao teclado. É retornado em AL 8 _flags_ diferentes, cada uma especificando informações diferentes sobre o estado atual do teclado. Conforme tabela:

| Bit | Flag |
| :--- | :--- |
| 0 | Tecla _shift_ direita está pressionada. |
| 1 | Tecla _shift_ esquerda está pressiona. |
| 2 | Tecla _ctrl_ está pressionada. |
| 3 | Tecla _alt_ está pressionada. |
| 4 | _Scroll lock_ está ligado. |
| 5 | _Num lock_ está ligado. |
| 6 | _Caps lock_ está ligado. |
| 7 | Modo _Insert_ está ligado. |

### Memória de Vídeo em _Text Mode_

Quando o sistema está em modo texto, a memória onde se armazena os caracteres começa no endereço 0xb800:0x0000 e ela é estruturada da seguinte forma:

```c
// Em modo de texto 80x25, padrão do MS-DOS

struct character {
    uint8_t ascii;
    uint8_t attribute;
};

struct character vmem[8][25][80];
```

Ou seja, começando em 0xb800:0x0000 as páginas estão uma atrás da outra na memória como uma grande _array_.

#### Atributo

O caractere nada mais é que o código ASCII do mesmo, já o atributo é um valor usado para especificar informações de cor e _blink_ do caractere. Podemos representar o valor em hexadecimal e desta forma o digito hexadecimal mais a direita seria referente ao atributo do texto, e o mais a esquerda referente ao atributo do fundo.

Os 4 bits \(_nibble_\) mais significativo indicam o atributo do fundo e os 4 bits menos significativos o atributo do texto, gerando uma cor na escala RGB. Caso não conheça, esta é a escala de cor da luz onde as cores primárias _Red_ \(vermelo\), _Green_ \(verde\) e _Blue_ \(azul\) são usadas em conjunto para formar qualquer outra cor. Conforme figura abaixo, podemos ver qual bit significa o quê:

![Bits de um atributo e seus significados](../.gitbook/assets/figura-atributos-de-caractere.png)

O atributo intensidade no atributo de texto, caso ligado, faz com que a cor do texto fique mais viva. Desligado as cores são mais escuras.  
Já o atributo _blink_ especifica se o texto deve permanecer piscando. Caso ativo, o texto irá ficar aparecendo e desaparecendo da tela constantemente.

### Olá Mundo

Um exemplo de "Hello World" usando alguns conceitos apresentados aqui.

```c
bits 16
org  0x100

%macro puts 2
  mov bx, %1
  mov bp, %2
  call puts
%endmacro


puts 0x000A, str1
puts 0x000C, str2

ret

str1: db `Hello World!\r\n`, 0
str2: db "Second message.", 0

; BL = Atributo
; BH = Página
; BP = ASCIIZ String
puts:
  mov ah, 0x03
  int 0x10

  mov  di, bp
  call strlen
  mov  cx, ax
  
  mov al, 0b01
  mov ah, 0x13
  int 0x10

  ret


; DI = ASCIIZ String
; Retorna o tamanho da string
strlen:
  mov cx, -1
  xor ax, ax

  repnz scasb

  mov ax, -2
  sub ax, cx
  ret
```

{% hint style="info" %}
Para uma lista completa de todas as interrupções definidas pelo BIOS, sugiro a leitura:

[http://vitaly\_filatov.tripod.com/ng/asm/asm\_001.html](http://vitaly_filatov.tripod.com/ng/asm/asm_001.html)
{% endhint %}

