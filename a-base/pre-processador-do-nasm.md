---
description: Aprendendo a usar o pré-processador do NASM
---

# Pré-processador do NASM

O NASM tem um pré-processador de código baseado no pré-processador da linguagem C. O que ele faz basicamente é interpretar instruções específicas do pré-processador para gerar o código fonte final, que será de fato compilado pelo NASM para o código de máquina. É por isso que tem o nome de **pré-**processador, já que ele processa o código fonte antes do NASM compilar o código.

As diretivas interpretadas pelo pré-processador são prefixadas pelo símbolo `%` e dão um poder absurdo para a programação diretamente em Assembly no nasm. Abaixo irei listar as mais básicas e o seu uso.

### %define

```text
%define nome              "valor"
%define nome(arg1, arg2)  arg1 + arg2
```

Assim como a diretiva `#define` do C, essa diretiva é usada para definir macros de uma única linha. Seja lá aonde o nome do macro for citado no código fonte ele expandirá para exatamente o conteúdo que você definiu para ele como se você estivesse fazendo uma cópia.

E assim como no C é possível passar argumentos para um macro usando de uma sintaxe muito parecida com uma função. Exemplo de uso:

```text
%define teste   mov eax, 31
teste
teste
```

As linhas 2 e 3 irão expandir para a instrução `mov eax, 31` como se tivesse feito uma cópia do valor definido para o macro. Podemos também é claro escrever um macro como parte de uma instrução, por exemplo:

```text
%define addr   [ebx*2 + 4]
mov eax, addr
```

Isso irá expandir a instrução na linha 2 para `mov eax, [ebx*2 + 4]`

A diferença entre definir um macro dessa forma e definir uma constante é que a constante recebe uma expressão matemática e expande para o valor do resultado. Enquanto o macro expande para qualquer coisa que você definir para ele.

O outro uso do macro, que é mais poderoso, é passando argumentos para ele assim como se é possível fazer em C. Para isso basta definir o nome do macro seguido dos parênteses e, dentro dos parênteses, os nomes dos argumentos que queremos receber separados por vírgula.

No valor definido para o macro os nomes desses argumentos irão expandir para qualquer conteúdo que você passe como argumento na hora que chamar um macro. Veja por exemplo o mesmo macro acima porém desta vez dando a possibilidade de escolher o registrador:

```text
%define addr(reg)   [reg*2 + 4]
mov eax, addr(ebx)
mov eax, addr(esi)
```

A linha 2 irá expandir para: `mov eax, [ebx*2 + 4]`.  
A linha 3 irá expandir para: `mov eax, [esi*2 + 4]`.

### %undef

```text
%undef nome_do_macro
```

Simplesmente apaga um macro anteriormente declarado por `%define`.

### %macro

```text
%macro nome NÚMERO_DE_ARGUMENTOS
  ; Código aqui
%endmacro
```

Além dos macros de uma única linha existem também os macros de múltiplas linhas que podem ser definidos no NASM.

Após a especificação do nome que queremos dar ao macro podemos especificar o número de argumentos passados para ele. Caso não queira receber argumentos no macro basta definir esse valor para zero. Exemplo:

```text
%macro sum5 0
  mov ebx, 5
  add eax, ebx
%endmacro

sum5
sum5
```

O `%endmacro` sinaliza o final do macro e todas as instruções inseridas entre as duas diretivas serão expandidas quando o macro for citado.

Para usar argumentos com um macro de múltiplas linhas difere de um macro definido com `%define`, ao invés do uso de parênteses o macro recebe argumentos seguindo a mesma sintaxe de uma instrução e separando cada um dos argumentos por vírgula. Para usar o argumento dentro do macro basta usar `%n`, onde **n** seria o número do argumento que começa contando em 1.

```text
%macro sum 2
  mov ebx, %2
  add %1, ebx
%endmacro

sum esi, edi
sum ebp, eax
```

Também é possível fazer com que o último argumento do macro expanda para todo o conteúdo passado, mesmo que contenha vírgula. Para isso basta adicionar um `+` ao número de argumentos. Por exemplo:

```text
%macro example 2+
  inc %1
  mov %2
%endmacro

example eax, ebx, ecx
example ebx, esi, edi, edx
```

A linha 6 expandiria para as instruções:

```text
inc eax
mov ebx, ecx
```

Enquanto a linha 7 iria acusar erro já que na linha 3 dentro do macro a instrução expandiu para `mov esi, edi, edx`, o que está errado.

É possível declarar mais de um macro com o mesmo nome desde que cada um deles tenham um número diferente de argumentos recebidos. O exemplo abaixo é totalmente válido:

```text
%macro example 1
  mov rax, %1
%endmacro

%macro example 2
  mov rax, %1
  add rax, %2
%endmacro


example 1
example 2, 3
```

#### Rótulos dentro de um macro

Usar um rótulo dentro de um macro é problemático porque se o macro for usado mais de uma vez estaremos redefinindo o mesmo rótulo já que seu nome nunca muda.

Para não ter esse problema existem os rótulos locais de um macro que será expandido para um nome diferente, definido pelo NASM, a cada uso do macro. A sintaxe é simples, basta prefixar o nome do rótulo com `%%`. Exemplo:

```text
; Repare como o código abaixo ficaria mais simples usando SETcc
; ou até mesmo CMOVcc.

%macro compare 2
  cmp %1, %2
  je %%is_equal
  mov eax, 0
  jmp %%end  
%%is_equal:
  mov eax, 1
%%end:
%endmacro

compare eax, edx
```

### %unmacro

```text
%unmacro nome NÚMERO_DE_ARGUMENTOS
```

Apaga um macro anteriormente definido com `%macro`. O número de argumentos especificado deve ser o mesmo utilizado na hora de declarar o macro.

### Compilação condicional

Assim como o pré-processador do C, o NASM também suporta diretivas de código condicional. A sintaxe básica é:

```text
%if<condição>
  ; Código 1
%elif<condição>
  ; Código 2
%else
  ; Código 3
%endif
```

Onde o código dentro da diretiva `%if` só é compilado se a condição for atendida. Caso não seja é possível usar a diretiva `%elif` para fazer o teste de uma nova condição. Enquanto o código na diretiva `%else` é expandido caso nenhuma das condições anteriormente testadas sejam atendidas. Por fim é usado a diretiva `%endif` para indicar o fim da diretiva `%if`.

É possível passar para `%if` e `%elif` uma expressão matemática afim de testar o resultado de um cálculo com uma constante ou algo semelhante. Se o valor for diferente de zero a expressão será considerada verdadeira e o bloco de código será expandido no código de saída.

Também é possível inverter a lógica das instruções adicionando um 'n', fazendo com que o bloco seja expandido caso a condição **não** seja atendida. Exemplo:

```text
CONST equ 5

%ifn CONST * 2 > 7
  call is_smallest
%else
  call is_bigger
%endif
```

Além do `%if` básico também podemos usar variantes que verificam por uma condição específica ao invés de receber uma expressão e testar seu resultado.

#### %ifdef e %elifdef

```text
%ifdef   nome_do_macro
%elifdef nome_do_macro
```

Essas diretivas verificam se um macro de linha única foi declarado por um `%define` anteriormente. É possível também usar essas diretivas em forma de negação adicionando o 'n' após o 'if'. Ficando: `%ifndef` e `%elifndef`, respectivamente.

#### %ifmacro e %elifmacro

```text
%ifmacro   nome_do_macro
%elifmacro nome_do_macro
```

Mesmo que `%ifdef` porém para macros de múltiplas linhas declarados por `%macro`. E da mesma que as diretivas anteriores também têm suas versões em negação: `%ifnmacro` e `%elifnmacro`.

### %error e %warning

```text
%error   "Mensagem de erro"
%warning "Mensagem de alerta"
```

Usando diretivas condicionais as vezes queremos acusar um erro ou emitir um alerta no console para indicar alguma mensagem no processo de compilação de algum projeto.

`%error` imprime a mensagem como um erro e finaliza a compilação, enquanto `%warning` emite a mensagem como um alerta e a compilação continua normalmente. Podemos por exemplo acusar um erro caso um determinado macro necessário para o código não esteja definido:

```text
%ifndef macro_importante
  %ifdef macro_substituto
    %warning "Macro importante não foi definido"
  %else
    %error "Macro importante e o seu substituto não foram definidos"
  %endif
%endif
```

### %include

```text
%include "nome do arquivo.ext"
```

Essa diretiva tem o uso parecido com a diretiva `#include` da linguagem C e ela faz exatamente a mesma coisa: Copia o conteúdo do arquivo passado como argumento para o exato local aonde ela foi utilizada no arquivo fonte. Seria como você manualmente abrir o arquivo, copiar todo o conteúdo dele e depois colar no código fonte.

Assim como fazemos em um _header file_ incluído por `#include` na linguagem C é importante usar as diretivas condicionais para evitar a inclusão duplicada de um mesmo arquivo. Por exemplo:

{% code title="arquivo.asm" %}
```text
%ifndef _ARQUIVO_ASM
%define _ARQUIVO_ASM

; Código aqui

%endif
```
{% endcode %}

Dessa forma quando incluirmos o arquivo pela primeira vez o macro `_ARQUIVO_ASM` será declarado. Se ele for incluído mais uma vez o macro já estará declarado e o `%ifndef` da linha 1 terá uma condição falsa e portanto não expandirá o conteúdo dentro de sua diretiva.

É importante fazer isso para evitar a redeclaração de macros, constantes ou rótulos. Bem como também evita que o mesmo código fique duplicado.

