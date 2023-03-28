---
description: >-
  Capítulo explicando os principais tópicos à respeito do Assembly e da
  arquitetura.
---

# A base

Para que fique mais prático para todos, independentemente se estiverem usando Linux/Windows/MacOS/BSD/etc, usaremos a linguagem C como "ambiente" para escrever código e podermos ver o resultado. Certifique-se de ter o [GCC](http://gcc.gnu.org/)/[Mingw-w64](http://mingw-w64.org/) e o [NASM](https://www.nasm.us/) instalados no seu sistema.

### Por que o GCC?

Caso você já programe em C e utilize outro compilador, mesmo assim recomendo que instale o GCC. O motivo disso é que irei ensinar o Inline Assembly deste compilador entre outras particularidades do mesmo. Também iremos analisar o código de saída do compilador, por isso é interessante que o código que você obter aí seja pelo menos parecido. Além disso também usaremos outras ferramentas do pacote GCC, como o **gdb** e o **ld** por exemplo.

### Por que o NASM?

O pacote GCC já tem o [assembler GAS](https://en.wikipedia.org/wiki/GNU\_Assembler) que é excelente mas prefiro usar aqui o NASM devido a vários fatores, dentre eles:

* O pré-processador do NASM é absurdamente incrível.
* O NASM tem uma sintaxe mais "legível" comparada a sintaxe do GAS.
* O NASM tem o ndisasm, vai ser útil na hora de estudar o código de máquina.
* Eu gosto do NASM.

Mais para frente no livro pretendo ensinar a usar o GAS também. Mas na base vamos usar só o NASM mesmo.

### Preparando o ambiente

Primeiramente eu recomendaria o uso de alguma distribuição Linux de 64-bit ou qualquer sistema operacional Unix-Like (\*BSD, MacOS etc). Isso porque mais para frente irei ensinar conteúdo que é exclusivo para sistemas operacionais compatíveis com o [UNIX](https://pt.wikipedia.org/wiki/Unix). Porém caso use o Windows não tem problema desde que instale o mingw-w64 como mencionei. O mais importante é ter um GCC que pode gerar código para 64-bit e 32-bit.

Vamos antes de mais nada preparar uma [PoC](https://pt.wikipedia.org/wiki/Prova\_de\_conceito) em C para chamar uma função escrita em Assembly. Este seria nosso arquivo **main.c**:

{% code title="main.c" %}
```c
#include <stdio.h>

int assembly(void);

int main(void)
{
  printf("Resultado: %d\n", assembly());
  return 0;
}
```
{% endcode %}

A ideia aqui é simplesmente chamar a função `assembly()` que iremos usar para testar algumas instruções escritas diretamente em Assembly. Ainda não aprendemos nada de Assembly então apenas copie e cole o código abaixo. Este seria nosso arquivo **assembly.asm**:

{% code title="assembly.asm" %}
```
bits 64

section .text

global assembly
assembly:
  mov eax, 777
  ret
```
{% endcode %}

No GCC você pode especificar se quer compilar código de 32-bit ou 64-bit usando a opção **-m** no Terminal. Por padrão o GCC já compila para 64-bit em sistemas de 64-bit. A opção **-c** no GCC serve para especificar que o compilador apenas faça o processo de compilação do código, sem fazer a ligação do mesmo. Deste jeito o GCC irá produzir um arquivo objeto como saída.

No nasm é necessário usar a opção **-f** para especificar o formato do arquivo de saída, no meu Linux eu usei -**f elf64** para especificar o formato de arquivo ELF. Caso use Windows então você deve especificar **-f win64**.

Por fim, para fazer a ligação dos dois arquivos objeto de saída podemos usar mais uma vez o GCC. Usar o ld diretamente exige incluir alguns arquivos objeto da libc, o que varia de sistema para sistema, portanto prefiro optar pelo GCC que irá por baixo dos panos rodar o ld incluindo os arquivos objetos apropriados. Para compilar e [linkar](https://pt.wikipedia.org/wiki/Ligador) os dois arquivos então fica da seguinte forma **no Linux**:

```
$ nasm assembly.asm -f elf64
$ gcc -c main.c -o main.o
$ gcc assembly.o main.o -o test -no-pie
$ ./test
```

No **Windows** fica assim:

```
$ nasm assembly.asm -f win64
$ gcc -c main.c -o main.o
$ gcc assembly.obj main.o -o test -no-pie
$ .\test
```

**Nota**: Repare que no Windows o nome padrão do arquivo de saída do nasm usa a extensão `.obj` ao invés de `.o`.

Usamos a opção **-o** no GCC para especificar o nome do arquivo de saída. E **-no-pie** para garantir que um [determinado recurso](https://en.wikipedia.org/wiki/Position-independent\_executable) do GCC não seja habilitado. O comando final acima seria somente a execução do nosso executável `test` em um sistema Linux. A execução do programa produziria o seguinte resultado no print abaixo, caso tudo tenha ocorrido bem.

![](<../.gitbook/assets/Captura de tela de 2019-07-16 10-47-32.png>)

{% hint style="info" %}
Mantenha essa PoC guardada no seu computador para eventuais testes. Você não será capaz de entender como ela funciona agora mas ela será útil para testar conceitos para poder vê-los na prática. Eventualmente tudo será explicado.
{% endhint %}

### Makefile

Caso você tenha o make instalado a minha recomendação é que organize os arquivos em uma pasta específica e use o **Makefile** abaixo.

{% code title="Makefile" %}
```
all:
	nasm *.asm -felf64
	gcc -c *.c
	gcc -no-pie *.o -o test
```
{% endcode %}

Isso é meio que gambiarra mas o importante agora é ter um ambiente funcionando.

### Se tudo deu errado...

Se você não conseguiu preparar nossa PoC aí no seu computador, acesse [o fórum do Mente Binária](https://www.mentebinaria.com.br/forums/) para tirar sua dúvida.
