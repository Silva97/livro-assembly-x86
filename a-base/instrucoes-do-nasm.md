---
description: Um pouco sobre o uso do nasm
---

# Instruções do nasm

Programar em Assembly é lidar diretamente com as instruções do arquivo binário final. Neste processo o assembler é o responsável por organizar o formato da saída e organizar o código.  
Vamos aprender aqui as diretivas e pseudo-instruções básicas do nasm para poder trabalhar melhor com nosso código.

### Seções

Ok, antes de mais nada vamos aprender a dividir nosso código em seções. Não adianta de nada usarmos um _linker_ se não trabalharmos com ele, não é mesmo?  
A sintaxe para definir uma seção é tão simples quanto possível, basta usar a diretiva `section` seguido do nome que você quer dar para a seção e os atributos que você quer definir para ela.  
As seções `.text`, `.data`, `.rodata` e `.bss` já tem seus atributos padrões definidos e por isso não precisamos setá-los.  
Por padrão o nasm joga todo o conteúdo do arquivo fonte na seção `.text` e por isso nós não a definimos na nossa PoC. Mas poderíamos reescrever nossa PoC desta vez especificando a seção:

{% tabs %}
{% tab title="assembly.asm" %}
```text
bits 64

section .text

global assembly
assembly:
  mov eax, 777
  ret
```
{% endtab %}

{% tab title="main.c" %}
```c
#include <stdio.h>

int assembly(void);

int main(void)
{
  printf("Resultado: %d\n", assembly());
  return 0;
}
```
{% endtab %}
{% endtabs %}

A partir da diretiva na linha 3, todo o código é organizado no arquivo objeto dentro da seção `.text`, que é destinada ao código executável do programa e por padrão tem o atributo de execução \(_exec_\) habilitado pelo nasm.

### Símbolos

Como já vimos na nossa PoC os símbolos internos podem ser exportados para poderem serem acessados a partir de outros arquivos objetos usando a diretiva `global`. Podemos exportar mais de um símbolo de uma vez separando cada nome de rótulo por vírgula, exemplo:

```text
global assembly, anotherFunction, gVariable
```

Deste jeito um endereço especificado por um rótulo no nosso código fonte em Assembly pode ser acessado por código fonte compilado em outro arquivo objeto, tudo graças ao _linker_.

Mas as vezes também teremos a necessidade de acessar um símbolo externo, isto é, pertencente a outro arquivo objeto.  
Para podermos fazer isso existe a diretiva `extern` que serve para declarar no arquivo objeto que estamos acessando um símbolo que está em outro arquivo objeto.  
Já vimos que no arquivo objeto **main.o** havia na _symbol table_ a declaração do uso do símbolo `assembly` que estava em um arquivo externo. A diretiva `extern` serve para inserir esta informação na tabela de símbolos do arquivo objeto de saída. A diretiva `extern` segue a mesma sintaxe de `global`:

```text
extern symbol1, symbol2, symbol3
```

Veja um exemplo de uso com nossa PoC:

{% tabs %}
{% tab title="main.c" %}
```c
#include <stdio.h>

int assembly(void);

int main(void)
{
  printf("Resultado: %d\n", assembly());
  return 0;
}

int number(void)
{
  return 777;
}
```
{% endtab %}

{% tab title="assembly.asm" %}
```
bits 64
extern number

section .text

global assembly
assembly:
  call number
  add eax, 111
  ret
```
{% endtab %}
{% endtabs %}

Declaramos na linha 11 do arquivo **main.c** a função `number`, e no arquivo **assembly.asm** usamos a diretiva `extern` na linha 2 para declarar o acesso ao símbolo `number`, que usamos na linha 8.

{% hint style="info" %}
Para o nasm não faz diferença alguma aonde você coloca as diretivas `extern` e `global`, porém por questões de legibilidade do código eu recomendo que use `extern` logo no começo do arquivo fonte e `global` logo antes da declaração do rótulo.  
Isto irá facilitar a leitura do seu código, já que ao ver o rótulo imediatamente se sabe que ele foi exportado...  
E ao abrir o arquivo fonte imediatamente, logo nas primeiras linhas, já se sabe quais símbolos externos estão sendo acessados.
{% endhint %}

### Variáveis?

Em Assembly não existe a declaração de uma variável, porém assim como funções existem como conceitos e podem ser implementados em Assembly, variáveis também são desta forma.

Em um código em C variáveis globais ficam na seção `.data` ou `.bss`.  
A seção `.data` do executável nada mais é que uma cópia dos dados contidos na seção `.data` do arquivo binário.  
Ou seja, o que despejarmos de dados em `.data` será copiado para a memória RAM e será acessível em tempo de execução e com permissão de escrita.  
Para despejar dados no arquivo binário existe a pseudo-instrução `db` e semelhantes, cada uma despejando um tamanho diferente de dados.  
Mas todas tendo a mesma sintaxe de separar cada valor numérico por vírgula.  
Veja a tabela:

| Pseudo-instrução | Tamanho dos dados | Bytes |
| :--- | :--- | :--- |
| db | byte | 1 |
| dw | word | 2 |
| dd | double word | 4 |
| dq | quad word | 8 |
| dt | ten word | 10 |
| do |  | 16 |
| dy |  | 32 |
| dz |  | 64 |

{% hint style="warning" %}
As quatro últimas `dt, do, dy e dz` não suportam que seja passado uma _string_ como valor.
{% endhint %}

Podemos por exemplo guardar uma variável global na seção .data e acessar ela a partir do código fonte em C, bem como também no próprio código em Assembly.  
Exemplo:

{% tabs %}
{% tab title="assembly.asm" %}
```text
bits 64

global myVar
section .data
  myVar: dd 777

section .text

global assembly
assembly:
  add dword [myVar], 3
  ret
```
{% endtab %}

{% tab title="main.c" %}
```c
#include <stdio.h>

int assembly(void);
extern int myVar;

int main(void)
{
  printf("Valor: %d\n", myVar);
  assembly();
  printf("Valor: %d\n", myVar);
  return 0;
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Repare que em C usamos a _keyword_ `extern` para especificar que a variável global `myVar` estaria em outro arquivo objeto, comportamento muito parecido com a diretiva `extern` do nasm.
{% endhint %}

### Variáveis não-inicializadas

A seção `.bss` é usada para armazenar variáveis não-inicializadas, isto é, que não tem um valor inicial definido.  
Basicamente esta seção no arquivo objeto tem um tamanho definido para ser alocada pelo sistema operacional em memória mas não um conteúdo explícito copiado do arquivo binário.  
Existem pseudo-instruções do nasm que permitem alocar espaço na seção sem de fato despejar nada ali.  
É a `resb` e suas semelhantes que seguem a mesma premissa de `db`.  
Os tamanhos disponíveis de dados são os mesmos de `db`, por isso não vou repetir a tabela aqui. Só ressaltando que a última letra da pseudo-instrução indica o tamanho do dado.  
A sintaxe da pseudo-instrução é:

```text
resb número_de_dados
```

Onde como operando ela recebe o número de dados do tamanho que a pseudo-instrução especifica serão alocados. Por exemplo:

```text
resd 6  ; Aloca o espaço de 6 double-words, ao todo 24 bytes.
```

A ideia de usar esta pseudo-instrução é poder declarar um rótulo/símbolo que irá apontar para o endereço dos dados alocados em memória.  
Veja mais um exemplo na nossa PoC:

{% tabs %}
{% tab title="assembly.asm" %}
```text
bits 64

global myVar
section .bss
  myVar: resd 1

section .text

global assembly
assembly:
  mov dword [myVar], 777
  ret
```
{% endtab %}

{% tab title="main.c" %}
```c
#include <stdio.h>

int assembly(void);
extern int myVar;

int main(void)
{
  assembly();
  printf("Valor: %d\n", myVar);
  return 0;
}
```
{% endtab %}
{% endtabs %}

### Constantes

Uma constante nada mais é que um apelido para representar um valor em código afim de facilitar a modificação daquele valor posteriormente ou então evitar um [_magic number_](https://pt.wikipedia.org/wiki/N%C3%BAmero_m%C3%A1gico_%28programa%C3%A7%C3%A3o_de_sistemas%29#N.C3.BAmeros_m.C3.A1gicos_em_programa.C3.A7.C3.A3o).  
Podemos declarar uma constante usando a pseudo-instrução `equ`:

```text
NOME_DA_CONSTANTE equ expressão
```

Por convenção é interessante usar nomes de constantes totalmente em letras maiúsculas para facilitar a sua identificação no código fonte em contraste com o nome de um rótulo.  
Seja lá aonde a constante for usada no código fonte, ela irá expandir para o seu valor definido no código binário resultante. Exemplo:

```text
EXAMPLE equ 34
mov eax, EXAMPLE
```

A instrução na linha 2 alteraria o valor de EAX para 34.

### Constantes em memória

Constantes em memória nada mais são do que valores despejados na seção `.rodata`, que é muito parecida com `.data` com a diferença de não ter permissão de escrita. Exemplo:

```text
section .rodata
  const_value: dd 777
```

### Expressões

O nasm aceita que você escreva expressões matemáticas seguindo a mesma sintaxe da linguagem C e seus operadores. Essas expressões serão calculadas pelo próprio nasm e não em tempo de execução, por isso é necessário adicionar a expressão somente rótulos, constantes ou qualquer outro valor que exista em tempo de compilação e não em tempo de execução.  
Podemos usar expressão matemática em qualquer pseudo-instrução ou instrução que aceita um valor numérico como operando. Exemplos:

```text
CONST equ (5 + 2*5) / 3       ; Correto!
mov eax, 4 << 2               ; Correto!
mov eax, [(2341 >> 6) % 10]   ; Correto!
mov eax, CONST + 4            ; Correto!

mov eax, ebx + 2   ; ERRADO!
```

O nasm também permite o uso de dois símbolos especiais que expandem para endereços relacionados a posição da instrução atual:

| Símbolo | Valor |
| :--- | :--- |
| $ | Endereço da instrução atual |
| $$ | Endereço do início da seção atual |



