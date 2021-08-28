---
description: Aprendendo a usar o depurador GDB do projeto GNU.
---

# Depurando com o GDB

O GDB é um depurador de linha de comando que faz parte do projeto GNU. O [Mingw-w64](https://mingw-w64.org/doku.php) já instala o GDB junto com o GCC, e no Linux ele pode ser instalado pelo pacote `gdb`:

```text
$ sudo apt install gdb
```

O GDB pode ser usado para depurar código tanto visualizando o Assembly como também o código-fonte. Para isso é necessário compilar o binário adicionando informações de depuração, com o GCC basta adicionar a opção `-g3` ao compilar. Exemplo:

```text
$ gcc -g3 test.c -o test
```

E pode rodar o GDB passando o caminho do binário assim:

```text
$ gdb ./test
```

{% hint style="info" %}
O caminho do binário é opcional. Caso especificado o GDB já inicia com esse binário como alvo para depuração, mas existem comandos do GDB que podem ser usados para escolher um alvo conforme será explicado mais abaixo.
{% endhint %}

O GDB funciona com comandos, quando você o inicia ele te apresenta um prompt onde você pode ir inserindo comandos para executar determinadas ações. Mais abaixo irei apresentar os principais comandos e como utilizá-los.

Esse depurador suporta depurar código de diversas linguagens de programação \(incluindo C++, Go e Rust\), mas aqui será demonstrado seu uso somente em um código escrito em C. O seguinte código será usado para demonstração:

{% code title="test.c" %}
```c
#include <stdio.h>

#define DEFINED_VALUE 12345

int add(int a, int b)
{
  return a + b;
}

int main(int argc, char **argv)
{
  char str[] = "a =";
  int x = 8;

  printf("%s %d\n", str, add(x, 3));
  return 0;
}
```
{% endcode %}

E será compilado da seguinte forma:

```text
$ gcc -g3 test.c -o test
```

{% hint style="info" %}
A opção `-g` é usada para adicionar informações de depuração ao executável. Esse `3` seria o nível de informações que serão adicionadas, onde 3 é o maior nível.

Para mais informações consulte a [documentação do GCC](https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html).
{% endhint %}

## Expressões

Determinadas instruções do GDB recebem uma expressão como argumento onde é possível usar qualquer tipo de constante, variável ou operador da linguagem que está sendo depurada \(neste caso C\). Isso inclui _casts_, _strings_ literais, macros e até mesmo chamadas de funções. Logo a expressão interpretada é quase idêntica a uma expressão que você escreveria na linguagem que está sendo depurada \(no nosso caso C\).

Também é possível referenciar o valor de algum registrador na expressão usando o prefixo `$`, como `$rax` por exemplo. Na imagem abaixo é uma demonstração usando o comando `print`:

![Sa&#xED;da do GDB ao usar o comando \`print\`](../.gitbook/assets/image%20%2813%29.png)

## Comandos

O GDB aceita abreviações dos comandos, onde ele identifica o comando a ser executado de acordo com suas primeiras letras ou abreviações definidas pelo depurador. Por exemplo o comando `breakpoint` pode ser executado também como `break`, `br` ou apenas `b`.

Ao apertar _enter_ sem digitar nenhum comando o GDB irá reexecutar o último comando que você executou.

### quit

```text
quit [EXPR]
```

Finaliza o GDB. A expressão opcional é avaliada e o resultado dela é usado como código de saída. Se a expressão não for passada o GDB sai com código `0`.

### file

```text
file FILE
```

Usa o arquivo binário especificado como alvo para depuração. O programa é procurado no diretório atual ou em qualquer caminho registrado na variável de ambiente PATH.

### attach e detach

```text
attach <process-id>
detach
```

O comando `attach` faz o [_attach_](entendendo-os-depuradores.md) no processo de ID especificado. Já o comando `detach` desfaz o _attach_ no processo que está atualmente conectado.

Você também pode iniciar a execução do GDB com a opção `-p` para ele já inicializar fazendo _attach_ em um processo, como em:

```text
$ gdb -p 12345
```

### breakpoint

```text
break [PROBE_MODIFIER] [LOCATION] [thread THREADNUM] [if CONDITION]
```

Se o comando for executado sem qualquer argumento o _breakpoint_ será adicionado na instrução atual.

LOCATION é a posição onde o _breakpoint_ deve ser inserido e pode ser o número de uma linha, endereço ou posição explícita.

Ao especificar o número da linha, o nome do arquivo e o número da linha são separados por `:`. Se não especificar o nome do arquivo o _breakpoint_ será adicionado a linha do arquivo atual. Exemplos:

```text
(gdb) b 15
(gdb) b test.c:17
```

Onde o primeiro adicionaria o _breakpoint_ na linha 15 do arquivo atual, e o segundo adicionaria na linha 17 do arquivo `test.c`.

O endereço pode ser simplesmente o nome de uma função ou então uma expressão, onde nesse caso é necessário usar `*` como prefixo ao símbolo ou endereço de memória. Como em:

```text
(gdb) b main
(gdb) b *main + 8
(gdb) b *0x12345
```

No primeiro caso um _breakpoint_ seria adicionado a função **main**. No segundo caso o endereço da primeira instrução da função **main** seria somado com 8, e o endereço resultante seria onde o _breakpoint_ seria inserido. Já no terceiro caso o _breakpoint_ seria inserido no endereço `0x12345`.

Também é possível especificar para qual _thread_ o _breakpoint_ deve ser inserido, onde por padrão o _breakpoint_ é válido para todas as _threads_. Exemplo:

```text
(gdb) b add thread 2
```

Isso adicionaria o _breakpoint_ somente para a _thread_ de ID 2.

{% hint style="info" %}
É possível usar o comando `info threads` para obter a lista de _threads_ e seus números de identificação.
{% endhint %}

E por fim dá para adicionar uma condição de parada ao _breakpoint_. Onde CONDITION é [uma expressão](depurando-com-o-gdb.md#expressoes) booleana. Exemplo:

```text
(gdb) b 7 if a == 8
```

Onde no contexto do nosso código de exemplo, `a` seria o primeiro parâmetro da função **add**.

### clear

```text
clear [LOCATION]
```

Remove um _breakpoint_ no local especificado. LOCATION funciona da mesma forma que no comando `breakpoint`.

Caso LOCATION não seja especificado remove o _breakpoint_ na posição atual.

### run

```text
run [arg1, arg2, arg3...]
```

O comando `run` inicia \(ou reinicia\) a execução do programa alvo. Opcionalmente pode-se passar argumentos de linha de comando para o programa. Caso os argumentos não sejam especificados, os mesmos argumentos utilizados na última execução de `run` serão utilizados.

Nos argumentos é possível usar o caractere curinga `*`, ele será expandido pela shell do sistema. Também é possível usar os redirecionadores `<`, `>` ou `>>`.

### kill

Finaliza a execução do programa que está sendo depurado.

### start, starti

```text
start [arg1, arg2, arg3...]
starti [arg1, arg2, arg3...]
```

O uso desses dois comandos é idêntico ao uso de `run`. Porém o comando `start` inicia a execução do programa parando no começo da função **main**. Já o `starti` inicia parando na primeira instrução do programa.

### next, nexti

```text
next [N]
nexti [N]
```

O comando `next` \(ou apenas `n`\) executa uma linha de código. Se N for especificado ele executa N linhas de código. Já o comando `nexti` \(ou apenas `ni`\) executa uma ou N instruções Assembly.

Os dois comandos atuam como um [_step over_](entendendo-os-depuradores.md#execucao-passo-a-passo), ou seja, não entram em chamadas de procedimentos.

### step, stepi

```text
step [N]
stepi [N]
```

O `step` \(ou `s`\) executa uma ou N linhas de código. Já o `stepi` \(ou `si`\) executa uma ou N instruções Assembly. Os dois comandos entram em chamadas de procedimentos.

### jump

```text
jump LOCATION
```

Salta \(modifica RIP\) para o ponto do código especificado. Onde LOCATION é idêntico ao caso do comando _breakpoint_ onde é possível especificar um número de linha ou endereço.

### advance

```text
advance LOCATION
```

Esse comando continua a execução do programa até o ponto do código especificado, daí para a execução lá. Assim como na instrução `jump`, o comando `advance` \(ou `adv`\) recebe um LOCATION como argumento.

O comando `advance` também para quando a função atual retorna.

### finish

Executa até o retorno da função atual. Quando a função retorna é criada uma variável \(como no caso do comando [print](depurando-com-o-gdb.md#print)\) com o valor de retorno da função.

### continue

Continua a execução normal do programa.

### record e reverse-\*

Imagine que mágico seria se o depurador pudesse voltar no tempo e desfazer as instruções executadas no programa, fazendo ele executar de maneira reversa parecido com rebobinar uma fita. Bom, o GDB pode fazer isso. 😎

Quando o programa já está em execução você pode executar o comando `record full` para iniciar a gravação das instruções executadas e `record stop` para parar de gravar.

Quando há a gravação é possível executar o programa em ordem reversa usando os comandos: `reverse-step` \(`rs`\), `reverse-stepi` \(`rsi`\), `reverse-next` \(`rn`\), `reverse-nexti` \(`rni`\) e `reverse-continue` \(`rc`\).

Esses comandos fazem a mesma coisa que os comandos normais, porém executando o programa ao reverso. Cada instrução revertida tem suas modificações na memória ou registradores desfeitas. Conforme demonstra a imagem abaixo.

![GDB usando execu&#xE7;&#xE3;o reversa.](../.gitbook/assets/image%20%2816%29.png)

Outros subcomandos de `record` são:

#### record goto

Salta para uma determinada instrução que foi gravada. Pode-se usar `record goto begin` para voltar ao início da gravação \(desfazendo todas as instruções\), `record goto end` para ir para o final da gravação ou `record goto N` onde N seria o número da instrução na gravação para saltar para ela.

#### record save &lt;filename&gt;

Salva os logs de execução no arquivo.

#### record restore &lt;filename&gt;

Restaura os logs de execução a partir do arquivo.

### thread

```text
thread <thread-id>
thread apply <thread-id> <command>
thread find <regex>
thread name <thread-name>
```

O comando `thread` pode ser usado para trocar entre _threads_ do processo. Você pode usar o comando `info threads` para listar as _threads_ do processo e obter seus ID. Exemplo:

```text
(gdb) thread 2
```

Isso trocaria para a _thread_ de ID 2. Esse comando também tem os seguintes subcomandos:

#### thread apply

Executa um comando na _thread_ especificada.

#### thread name

Define um nome para a _thread_ atual, facilitando a identificação dela.

#### thread find

Recebe uma expressão regular como argumento que é usada para listar as _threads_ cujo o nome coincida com a expressão regular. O comando exibe o ID das _threads_ listadas.

### print

```text
print[/FMT] [EXPR]
```

O comando `print` \(ou `p`\) exibe no terminal o resultado da expressão passada como argumento. Opcionalmente pode-se especificar o formato de saída, onde os formatos são os mesmos utilizados no [comando x](depurando-com-o-gdb.md#x). Exemplo:

```text
(gdb) p/x 15
$1 = 0xf
```

Repare que a cada execução do comando `print` ele define uma variável \(`$1`, `$2` etc.\) que armazena o resultado da expressão do comando. Você também pode usar o valor dessas variáveis em uma expressão e assim reaproveitar o resultado de uma execução anterior do comando. Os símbolos `$` e `$$` se referem aos valores da última e penúltima execução do comando, respectivamente. Exemplo:

```text
(gdb) p x + $3
```

Existe também o operador binário `@` que pode ser usado para tratar o valor no endereço especificado como uma _array_. O formato do uso desse operador é `array@size`, passando à esquerda o primeiro elemento da _array_.

Onde o tipo de cada elemento da _array_ é definido de acordo com o tipo do objeto que está sendo referenciado. Na imagem abaixo é demonstrado o uso desse operador para visualizar todo o conteúdo da _array_ **argv**.

![Sa&#xED;da do GDB ao usar Artificial Array](../.gitbook/assets/image%20%2812%29.png)

### printf

```text
printf "format string", ARG1, ARG2, ARG3, ..., ARG
```

Esse comando pode ser usado de maneira semelhante a função **printf** da libc. Cada argumento é separado por vírgula e o primeiro argumento é a _format string_ que suporta quase todos os formatos suportados pela função **printf**. Os demais argumentos são [expressões](depurando-com-o-gdb.md#expressoes).

Exemplo de uso:

```text
(gdb) printf "%p\n", $rsp
0x7fffffffdf20
```

### dprintf

```text
dprintf LOCATION, "format string", ARG1, ARG2, ARG3, ..., ARG
```

Esse comando insere um _breakpoint_ no código onde, toda vez que ele é alcançado, o comando `printf` é executado e depois a execução continua. O uso desse comando é semelhante ao do comando `printf`. Exemplo:

```text
(gdb) dprintf 7, "%d + %d\n", a, b
```

No nosso código de exemplo, isso inseria o _dynamic printf_ na linha 7 que está dentro da função **add**. Conforme a imagem abaixo demonstra:

![Demonstra&#xE7;&#xE3;o de uso do dprintf no GDB](../.gitbook/assets/image%20%2815%29.png)

### x

```text
x[/FMT] ADDRESS
```

O comando `x` serve para ver valores na memória. O argumento FMT \(opcional\) é o número de valores a serem exibidos, seguido de uma letra indicando o formato do valor seguido de uma letra que indica o tamanho do valor. Por padrão exibe apenas um valor caso o número não seja especificado. O formato e tamanho padrão é o mesmo utilizado na última execução do comando `x`.

As letras de formato são: `o` \(octal\), `x` \(hexadecimal\), `d` \(decimal\), `u` \(decimal não-sinalizado\), `t` \(binário\), `f` \(_float_\), `a` \(endereço\), `i` \(instrução\), `c` \(caractere de 1 byte\), `s` \(_string_\) e `z` \(hexadecimal com zeros à esquerda\).

Ao usar o formato `i` será feito o _disassembly_ do código no endereço. O número de valores é usado para especificar o número de instruções para fazer o _disassembly_.

Exemplo:

```text
(gdb) x/x 0x7fffffffdf64
0x7fffffffdf64:	0x003d2061
```

As letras de tamanho são: `b` \(byte\), `h` \(metade de uma palavra\), `w` \(palavra\) e `g` \(_giant_, 8 bytes\). Na arquitetura x86-64 uma palavra é 32-bit \(4 bytes\).

Exemplos:

```text
(gdb) x/xb 0x7fffffffdf64
0x7fffffffdf64:	0x61
(gdb) x/4xb 0x7fffffffdf64
0x7fffffffdf64:	0x61	0x20	0x3d	0x00
```

### disassembly

```text
disassembly[/MODIFIER] [ADDRESS]
disassembly[/MODIFIER] start,end
disassembly[/MODIFIER] start,+length
```

O comando `disassembly` \(ou `disas`\) pode ser usado para exibir o _disassembly_ de uma função ou _range_ de endereço. O argumento ADDRESS \(opcional\) é uma expressão, sem esse argumento ele faz o _disassembly_ na posição ou função atual.

Também é possível especificar um _range_ de endereços para exibir o _dissasembly_ das instruções, separando o endereço inicial e final por vírgula. Se usar o `+` no segundo argumento separado por vírgula, ele é considerado como o tamanho em bytes do _range_ iniciado em **start**.

Exemplos:

```text
(gdb) disas 0x00005555555551b4,0x00005555555551b9
Dump of assembler code from 0x5555555551b4 to 0x5555555551b9:
   0x00005555555551b4 <main+51>:	mov    $0x3,%esi
End of assembler dump.
(gdb) disas 0x00005555555551b4,+5
Dump of assembler code from 0x5555555551b4 to 0x5555555551b9:
   0x00005555555551b4 <main+51>:	mov    $0x3,%esi
End of assembler dump.
```

O argumento MODIFIER é uma \(ou mais\) das seguintes letras:

* `s` - Exibe também as linhas de código correspondentes as instruções em Assembly.
* `r` - Também exibe o código de máquina em hexadecimal.

Exemplo:

```text
(gdb) disas/rs 0x00005555555551b4,+5
Dump of assembler code from 0x5555555551b4 to 0x5555555551b9:
test.c:
15	  printf("%s %d\n", str, add(x, 3));
   0x00005555555551b4 <main+51>:	be 03 00 00 00	mov    $0x3,%esi
End of assembler dump.
```

{% hint style="info" %}
Por padrão o _disassembly_ é feito em sintaxe AT&T, mas você pode modificar para sintaxe Intel com o comando: `set disassembly-flavor intel`
{% endhint %}

### list

```text
list
list LINENUM
list FILE:LINENUM
list FUNCTION
list FILE:FUNCTION
list *ADDRESS
```

Exibe a listagem de código na linha ou início da função especificada. Um endereço também pode ser especificado usando um `*` como prefixo, as linhas de código correspondentes ao endereço serão exibidas.

Caso `list` seja executado sem argumentos mais linhas são exibidas a partir da última linha exibida pela última execução de `list`.

O número de linhas exibido é por padrão 10, mas esse valor pode ser alterado com o comando `set listsize <number-of-lines>`.

### backtrace

```text
backtrace [COUNT]
```

O comando `backtrace` \(ou `bt`\) exibe o _stack backtrace_ atual. O argumento COUNT é o número máximo de _stack frames_ que serão exibidos. Se for um número negativo exibe os primeiros _stack frames_.

Exemplo:

```text
(gdb) bt
#0  add (a=8, b=3) at test.c:7
#1  0x00005555555551c0 in main (argc=1, argv=0x7fffffffe068) at test.c:15
```

### frame

```text
frame [FRAME_NUMBER]
```

Sem argumentos exibe o _stack frame_ selecionado. Caso seja especificado um número como argumento, seleciona e exibe o _stack frame_ indicado pelo número. Esse número pode ser consultado com o comando `backtrace`.

Esse comando tem os seguintes subcomandos:

#### frame address

```text
frame address STACK_ADDRESS
```

Exibe o _stack frame_ no endereço especificado.

#### frame apply

```text
frame apply COUNT COMMAND
frame apply all COMMAND
frame apply level FRAME_NUMBER COMMAND
```

O comando `frame apply` executa o mesmo comando em um ou mais _stack frames_. Esse subcomando é útil, por exemplo, para ver o valor das variáveis locais que estão em uma função de outro _stack frame_ além do atual.

COUNT é o número de _frames_ onde o comando será executado. Por exemplo `frame apply 2 p x` executaria o comando `print` nos últimos 2 _frames_ \(o atual e o anterior\).

 O `frame apply all` executa o comando em todos os _frames_. Já o `frame apply level` executa o comando em um _frame_ específico. exemplo:

```text
(gdb) frame apply level 1 info locals
#1  0x00005555555551c0 in main (argc=1, argv=0x7fffffffe068) at test.c:15
str = "a ="
x = 8
```

#### frame function

```text
frame function FUNCTION_NAME
```

Exibe o _stack frame_ da função especificada.

#### frame level

```text
frame level FRAME_NUMBER
```

Exibe o _stack frame_ do número especificado.

### info

O comando `info` contém diversos subcomandos para exibir informações sobre o programa que está sendo depurado. Abaixo será listado apenas os subcomandos principais.

#### info registers

Exibe os valores dos registradores. Pode-se passar como argumento uma lista \(separada por espaço\) dos registradores para exibir. Sem argumentos exibe o valor de todos os [registradores de propósito geral](../a-base/registradores-gerais.md), [registradores de segmento](../aprofundando-em-assembly/registradores-de-segmento.md) e [EFLAGS](../aprofundando-em-assembly/flags-do-processador.md). Exemplo:

```text
(gdb) info reg rax rbx
```

#### info frame

O uso desse subcomando é semelhante ao uso do comando [frame](depurando-com-o-gdb.md#frame) e contém os mesmos subcomandos. A diferença é que ele exibe todas as informações relacionadas ao _stack frame_. Enquanto o comando `frame` apenas exibe informações de um ponto de vista de alto-nível.

#### info args

```text
info args [NAMEREGEXP]
```

Exibe os argumentos passados para a função do _stack frame_ atual. Se NAMEREGEXP for especificado exibe apenas os argumentos cujo o nome coincida com a expressão regular.

#### info locals

```text
info locals [NAMEREGEXP]
```

Uso idêntico ao de `info args` só que exibe o valor das variáveis locais.

#### info functions

```text
info functions [NAMEREGEXP]
```

Exibe todas as funções cujo o nome coincida com a expressão regular. Se o argumento não for especificado lista todas as funções.

#### info breakpoints

Exibe os _breakpoints_ definidos no programa.

#### info source

Exibe informações sobre o código-fonte atual.

#### info threads

Lista as _threads_ do processo.

### display e undisplay

```text
display[/FMT] EXPRESSION
undisplay [NUM]
```

Esse comando pode ser usado da mesma maneira que o comando [print](depurando-com-o-gdb.md#print). Ele registra uma expressão para ser exibida a cada vez que a execução do processo faz uma parada. Exemplo:

```text
(gdb) display/7i $rip
```

Isso exibiria o _disassembly_ de 7 instruções a partir de RIP a cada passo executado.

Se `display` for executado sem argumentos ele exibe todas as expressões registradas para auto-display.

Enquanto o comando `undisplay` remove a expressão com o número especificado. Sem argumentos remove todas as expressões registradas por `display`.

### source

```text
source FILE
```

Carrega o arquivo especificado e executa os comandos no arquivo como um script.

{% hint style="info" %}
Quando o GDB inicia ele faz o _source_ automático do script de nome `.gdbinit` presente na sua pasta _home_. Exceto se o GDB for iniciado com a _flag_ `--nh`.
{% endhint %}

### help

O comando `help`, sem argumentos, lista as classes de comandos. É possível rodar `help CLASS` para obter a lista de comandos daquela classe.

Também é possível rodar `help COMMAND` para obter ajuda para um comando específico, pode-se inclusive usar abreviações. E também é possível obter ajuda para subcomandos, conforme exemplos:

```text
(gdb) help ni
(gdb) help info reg
(gdb) help frame apply level
```

## Text User Interface \(TUI\)

É possível usar o GDB com uma interface textual permitindo que seja mais agradável acompanhar a execução enquanto observa o código-fonte. Para isso basta iniciar o GDB com a _flag_ `-tui`, como em:

```text
$ gdb -tui ./test
```

![GDB Text User Interface](../.gitbook/assets/image%20%2814%29.png)

### Atalhos de teclado

| Atalho de teclado | Descrição |
| :--- | :--- |
| Ctrl+x a | O atalho `Ctrl+x a` \(Ctrl+x seguido da tecla `a`\) alterna para o modo TUI caso tenha iniciado o GDB normalmente. |
| Ctrl+x 1 | Alterna para o _layout_ de janela única. |
| Ctrl+x 2 | Alterna para o _layout_ de janela dupla. Quando já está no _layout_ de janela dupla o próximo _layout_ com duas janelas é selecionado. Onde é possível exibir código-fonte+Assembly, registradores+Assembly e registradores+código-fonte. |
| Ctrl+x o | Muda a janela ativa. |
| Ctrl+x s | Muda para o modo [Single Key Mode](depurando-com-o-gdb.md#single-key-mode). |
| PgUp | Rola a janela ativa uma página para cima. |
| PgDn | Rola a janela ativa uma página para baixo. |
| ↑ \(Up\) | Rola a janela ativa uma linha para cima. |
| ↓ \(Down\) | Rola a janela ativa uma linha para baixo. |
| ← \(Left\) | Rola a janela ativa uma coluna para a esquerda. |
| → \(Right\) | Rola a janela ativa uma coluna para a direita. |
| Ctrl+L | Redesenha a tela. |

### Single Key Mode

Quando se está no modo _**Single Key**_ é possível executar alguns comandos pressionando uma única tecla,  conforme tabela abaixo:

| Tecla | Comando | Nota |
| :--- | :--- | :--- |
| c | continue |  |
| d | down |  |
| f | finish |  |
| n | next |  |
| o | nexti | "o" de _step **o**ver_. |
| q | - | Sai do modo _Single Key_. |
| r | run |  |
| s | step |  |
| i | stepi |  |
| u | up |  |
| v | info locals | "v" de _**v**ariables_. |
| w | where | Alias para o comando `backtrace`. |

Qualquer outra tecla alterna temporariamente para o modo de comandos. Após um comando ser executado ele retorna para o modo _Single Key_.

