---
description: Chamada de sistema no Linux
---

# Syscall no Linux

Uma chamada de sistema, ou _syscall_ \(abreviação para _system call_\), é algo muito parecido com uma `call` mas com a diferença nada sutil de que é o kernel do sistema operacional quem irá executar o código.

O kernel é a parte principal de um sistema operacional encarregada de gerenciar todo o sistema, desde o hardware até mesmo a execução do software \(processos/tarefas\). Ele é a base de todo o restante do sistema que roda sob controle do kernel. O Linux na verdade é um kernel, um "sistema operacional Linux" na verdade é um sistema operacional que usa o kernel Linux.

Em x86-64 existe uma instrução que foi feita especificamente para fazer chamadas de sistema e o nome dela é, intuitivamente, `syscall`. Ela não recebe nenhum operando e a especificação de qual código ela irá executar e com quais argumentos é definido por uma convenção de chamada assim como no caso das funções.

### Convenção de syscall x86-64

A convenção para efetuar uma chamada de sistema em Linux x86-64 é bem simples, basta definir RAX para o número da syscall que você quer executar e outros 6 registradores são usados para passar argumentos. Veja a tabela:

| Registrador | Uso |
| :--- | :--- |
| RAX | Número da syscall / Valor de retorno |
| RDI | 1° argumento |
| RSI | 2° argumento |
| RDX | 3° argumento |
| R10 | 4° argumento |
| R8 | 5° argumento |
| R9 | 6° argumento |

O retorno da _syscall_ também fica em RAX assim como na convenção de chamada da linguagem C.

{% hint style="info" %}
Em _syscalls_ que recebem menos do que 6 argumentos não é necessário definir o valor dos registradores restantes porque não serão utilizados.
{% endhint %}

### exit

| Nome | RAX | RDI |
| :--- | :--- | :--- |
| exit | 60 | int status\_de\_saída |

Vou ensinar aqui a usar a _syscall_ mais simples que é a `exit`, ela basicamente finaliza a execução do programa. Ela recebe um só argumento que é o status de saída do programa. Esse número nada mais é do que um valor definido para o sistema operacional que indica as condições da finalização do programa.

Por convenção geralmente o número zero indica que o programa finalizou sem problemas, e qualquer valor diferente deste indica que houve algum erro. Um exemplo na nossa PoC:

{% tabs %}
{% tab title="assembly.asm" %}
```text
bits 64

section .text

global assembly
assembly:
  mov rax, 60
  mov rdi, 0
  syscall
  ret
```
{% endtab %}

{% tab title="main.c" %}
```c
#include <stdio.h>

int assembly(void);

int main(void)
{
  assembly();
  puts("Hello World!");
  return 0;
}
```
{% endtab %}
{% endtabs %}

A instrução `ret` na linha 10 nunca será executada porque a _syscall_ disparada pela instrução `syscall` na linha 9 não retorna. No momento em que for chamada o programa será finalizado com o valor de RDI como status de saída.

No Linux se quiser ver o status de saída de um programa a variável especial `$?` expande para o status de saída do último programa executado. Então você pode executar nossa PoC da seguinte forma:

```text
$ ./test
$ echo $?
```

O `echo` teoricamente iria imprimir **0** que é o status de saída que nós definimos. Experimente mudar o valor de RDI e ver se reflete na mudança do valor de  `$?` corretamente.

### Outras syscalls

Se quiser ver uma lista completa de _syscalls_ x86-64 do Linux pode ver no link abaixo:

* [Linux System Call Table for x86 64](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)

Você também pode consultar o conteúdo do arquivo cabeçalho `/usr/include/x86_64-linux-gnu/asm/unistd_64.h` para ver uma lista completa da definição dos números de syscall.

Além disso também sugiro consultar a _man page_ do [wrapper](https://pt.wikipedia.org/wiki/Wrapper) em C da syscall afim de entender mais detalhadamente o que cada uma delas faz. Por exemplo:

```text
$ man 2 exit
```

E para simplificar a consulta de syscalls no meu Linux eu implementei e uso a seguinte função em Bash. Fique à vontade para usá-la:

```bash
function syscall() {
	if [ -z "$1" ]; then
		echo "Developed by Luiz Felipe <felipe.silva337@yahoo.com>"
		echo "See (Portuguese): https://mentebinaria.gitbook.io/assembly-x86"
		echo
		echo "Usage: syscall name [32|64]"
		return 0
	fi

	name="$1"
	bits="${2-64}"
	number=$(grep -m1 "__NR_$name" \
			 "/usr/include/x86_64-linux-gnu/asm/unistd_$bits.h" \
			 | cut -d' ' -f3)

	[ -z "$number" ] && return 1

	if [ "$bits" == "64" ]; then
		sysnumRegister="RAX"
		arguments="RDI, RSI, RDX, R10, R8, R9"
	else
		sysnumRegister="EAX"
		arguments="EBX, ECX, EDX, ESI, EDI, EBP"
	fi

	echo "Syscall number ($sysnumRegister): $number"
	echo "Arguments: $arguments"
	echo
	echo "Synopsis:"

	awkCode='
		/SYNOPSIS/,/DESCRIPTION/{
			if ($1 != "SYNOPSIS" && $1 != "DESCRIPTION") {
				print $0
			}
		}
	'

	man 2 "$name" | awk "$awkCode"

	return 0
}
```

Exemplo de uso:

![](../.gitbook/assets/image%20%287%29.png)

