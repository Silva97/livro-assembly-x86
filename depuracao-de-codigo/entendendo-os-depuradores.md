---
description: Entendendo os conceitos principais sobre um depurador e como eles funcionam.
---

# Entendendo os depuradores

Depuradores (_debuggers_) são ferramentas que atuam se conectando (_attaching_) em processos para controlar e monitorar a execução dos mesmos. Isso é possível por meio de recursos que o próprio sistema operacional provém, no caso do Linux por meio da _syscall_ **ptrace**.

O processo que se conecta é chamado de _tracer_, e o processo conectado é chamado de _tracee_. Essa conexão é chamada de _attach_ e é feita em uma _thread_ individual do processo. Quando o depurador faz _attach_ em um processo ele, na verdade, está fazendo _attach_ na _thread_ principal do processo.

## Processos

{% hint style="info" %}
As _threads_ são tarefas individuais em um processo. Cada _thread_ de um processo executa um código diferente de maneira concorrente em relação as outras _threads_ do mesmo processo.
{% endhint %}

Um processo é basicamente a imagem de um programa em execução. Uma parte do sistema operacional conhecida como _loader_ (ou _dynamic linker_) é a responsável por ler o arquivo executável, mapear seus códigos e dados na memória, carregar dependências (bibliotecas) resolvendo seus símbolos e iniciar a execução da _thread_ principal do processo no código que está no endereço do _entry point_ do executável. Onde _entry point_ se trata de um endereço armazenado dentro do arquivo executável e é o endereço onde a _thread_ principal inicia a execução.

O depurador tem acesso a memória de um processo e pode controlar a execução das _threads_ do processo. Ele também tem acesso a outras informações sobre o processo, como o valor dos registradores em uma _thread_ por exemplo.

### Context switch

Do ponto de vista de cada _thread_ de um processo ela tem exclusividade na execução de código no processador e no acesso a seus recursos. Inclusive em Assembly usamos registradores do processador diretamente sem nos preocuparmos com outras _threads_ (do mesmo processo ou de outros) usando os mesmos registradores "ao mesmo tempo".

Cada núcleo (_core_) do processador tem um conjunto individual de registradores, mas é comum em um sistema operacional moderno diversas tarefas estarem concorrendo para executar em um mesmo núcleo.

Uma parte do sistema operacional chamada de _scheduler_ é responsável por gerenciar quando e qual tarefa será executada em um determinado núcleo do processador. Isso é chamado de escalonamento de processos (_scheduling_) e quando o _scheduler_ suspende a execução de uma tarefa para executar outra isso é chamado de **troca de contexto** ou **troca de tarefa** (_context switch_ ou _task switch_).

Quando há a troca de contexto o _scheduler_ se encarrega de salvar na memória RAM o estado atual do processo, e isso inclui o valor dos registradores. Quando a tarefa volta a ser executada o estado é restaurado do ponto onde ele parou, e isso inclui restaurar o valor de seus registradores.

É assim que cada _thread_ tem valores distintos em seus registradores. É assim também que depuradores são capazes de ler e modificar o valor de registradores em uma determinada _thread_ do processo, o sistema operacional dá a capacidade de acessar esses valores no contexto da tarefa e permite fazer a modificação. Quando o _scheduler_ executar a tarefa o valor dos registradores serão atualizados com o valor armazenado no contexto.

{% hint style="info" %}
Processadores Intel mais modernos têm uma tecnologia chamada **Hyper-Threading**. Essa tecnologia permite que um mesmo núcleo atue como se fosse dois permitindo que duas _threads_ sejam executadas paralelamente no mesmo núcleo.

Cada "parte" independente é chamada de processador lógico (_logical processor_) e cada processador lógico no núcleo tem seu conjunto individual de registradores. Com exceção de alguns registradores "obscuros" que são compartilhados pelos processadores lógicos do núcleo. Esses registradores não foram abordados no livro, mas caso esteja curioso pesquise por _Model-specific register_ (MSR) e MTRRs. Apenas alguns MSR são compartilhados pelos processadores lógicos.
{% endhint %}

### Sinais

Os sinais é um mecanismo de comunicação entre processos (_Inter-Process Communication_ - IPC). Existem determinados sinais em cada sistema operacional e quando um sinal é enviado para um processo esse processo é temporariamente suspenso e um tratador (_handler_) do sinal é executado.

A maioria dos sinais podem ter o tratador personalizado pelo programador mas alguns têm um tratador padrão e não podem ser alterados. É o caso por exemplo no Linux do sinal SIGKILL, que é o sinal enviado para um processo quando você tenta forçar a finalização dele (com o comando `kill -9` por exemplo). O tratador desse sinal é exclusivamente controlado pelo sistema operacional e o processo não é capaz de personalizar ele.

Exemplo de personalização do tratador de um sinal:

{% code title="main.c" %}
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

void termination(int signum)
{
  puts("Goodbye!");
  exit(EXIT_SUCCESS);
}

int main(void)
{
  struct sigaction action = {
      .sa_handler = termination,
  };

  sigaction(SIGTERM, &action, NULL);
  int pid = getpid();

  printf("My PID: %d\nPlease, send-me SIGTERM... ", pid);
  while (1)
    getchar();

  return 0;
}
```
{% endcode %}

Experimente compilar e executar esse programa. No Linux você pode enviar o sinal SIGTERM para o processo com o comando `kill`, como em:

```bash
$ kill 155541

# Onde 155541 seria o PID do processo.
```

O sinal SIGTERM seria o jeito "educado" de finalizar um processo. Porém como pode ser observado é possível que o processo personalize o tratador desse sinal, que por padrão finaliza o programa. Nesse código de exemplo se removermos a chamada para a função **exit** o processo não irá mais finalizar ao receber SIGTERM. É por isso que existe o sinal mais "invasivo" SIGKILL que foi feito para ser usado quando o processo não está mais respondendo.

Um processo que está sendo depurado (o _tracee_) para toda vez que recebe um sinal e o depurador toma o controle da execução. Exceto no caso de SIGKILL que funciona normalmente sem a intervenção do depurador.

### Exceções

Quando um processo dispara uma [exceção](../aprofundando-em-assembly/interrupcoes-de-software.md), um tratador (_handler_) configurado pelo sistema operacional envia um sinal para o processo tratar aquela exceção. Depuradores são capazes de identificar (e ignorar) exceções intervindo no processo de _handling_ desse sinal.

## Depuradores

Agora que já entendemos um pouco sobre processos vai ficar mais fácil entender como depuradores funcionam. Afinal de contas depuradores depuram processos. :slight\_smile:&#x20;

Depuradores têm a capacidade de controlar a execução das _threads_ de um processo, tratar os sinais enviados para o processo, acessar sua memória e ver/editar dados relacionados ao contexto de cada _thread_ (como os registradores, por exemplo). Todo esse poder é dado para os usuários do depurador por meio de alguns recursos que serão descritos abaixo.

### Breakpoint

Um ponto de parada (_breakpoint_) é um ponto no código onde a execução do programa será interrompida e o depurador irá manter o programa em pausa para que o usuário possa controlar a execução em seguida.

Os _breakpoints_ são implementados na prática (na arquitetura x86-64) como uma instrução `int3` que dispara [a exceção](../aprofundando-em-assembly/interrupcoes-de-software.md) #BP. Quando um depurador insere um _breakpoint_ em um determinado ponto do código ele está simplesmente modificando o primeiro byte da instrução para o byte **0xCC**, que é o byte da instrução `int3`. Quando a exceção é disparada o sinal SIGTRAP é enviado para o processo e o depurador se encarrega de dar o controle da execução para o usuário. Quando o usuário continua a execução o depurador restaura o byte original da instrução, executa ela e coloca o byte **0xCC** novamente.

{% hint style="info" %}
Em arquiteturas que não têm uma exceção específica para disparar _breakpoints_ os depuradores substituem a instrução por alguma outra instrução que disparará alguma exceção. Como uma instrução de divisão ilegal por exemplo.
{% endhint %}

Podemos comprovar isso com o seguinte código:

```c
#include <stdio.h>
#include <signal.h>

void breakpoint(int signum)
{
  puts("Breakpoint!");
}

int main(void)
{
  struct sigaction action = {
      .sa_handler = breakpoint,
  };

  sigaction(SIGTRAP, &action, NULL);

  asm("int3");
  puts("...");

  return 0;
}
```

Ao executar a instrução `int3` inserida com [inline Assembly](../programando-junto-com-c/inline-assembly-no-gcc.md) na linha 17, o processo recebe o sinal SIGTRAP e nosso tratador é executado. Experimente comentar a chamada para **sigaction** na linha 15 para ver o resultado do tratador padrão.

### Hardware/Software breakpoint

O termo _software breakpoint_ é usado para se referir a um _breakpoint_ que é definido e configurado por software (o depurador), como o que já foi descrito acima. Por exemplo _breakpoints_ podem ter uma condição de parada e isso é implementado pelo próprio depurador. Ele faz o tratamento do _breakpoint_ normalmente mas antes verifica a condição, se a condição não for atendida ele continua a execução do código como se o _breakpoint_ nunca tivesse acontecido.

Já o termo _hardware breakpoint_ é usado para se referir a um _breakpoint_ que é suportado pelo próprio processador. A arquitetura x86-64 tem 8 registradores de depuração (_debug registers_) onde 4 deles podem ser usados para indicar _breakpoints_.

Os registradores DR0, DR1, DR2 e DR3 armazenam o endereço onde irá ocorrer o _breakpoint_. Já o registrador DR7 habilita ou desabilita esses _breakpoints_ e configura uma condição para eles. Onde a condição determina em qual ocasião o _breakpoint_ será disparado, como por exemplo ao ler/escrever naquele endereço ou ao executar a instrução no endereço.

Quando a condição do _breakpoint_ é atendida o processador dispara [uma exceção](../aprofundando-em-assembly/interrupcoes-de-software.md) #BP.

{% hint style="warning" %}
Os _debug registers_ não podem ser lidos/modificados sem privilégios de kernel. Rodando sobre um sistema operacional um processo comum não é capaz de manipulá-los diretamente.
{% endhint %}

Esse mesmo recurso (com até mais recursos ainda) poderia ser implementado pelo depurador com um _software breakpoint_. Por exemplo caso o depurador queira que um _breakpoint_ seja disparado ao ler/escrever em um determinado endereço o depurador pode simplesmente modificar as permissões de acesso daquele endereço e, quando o processo fosse acessar os dados naquele endereço, uma exceção #GP seria disparada e o depurador poderia retomar o controle da execução.

### Execução passo-a-passo

Depuradores não são apenas capazes de executar o software e esperar por um _breakpoint_ para retomar o controle. Eles podem também executar apenas uma instrução da _thread_ por vez e permanecer controlando a execução. Isso é chamado de execução passo-a-passo (_step-by-step_), onde o "passo" é uma única instrução. O usuário do depurador pode clicar em um botão ou executar um comando e apenas uma instrução do processo será executada, e o usuário pode ver o resultado da instrução e optar pelo que fazer em seguida.

Isso é implementado na arquitetura x86-64 usando a _trap flag_ (TF) no registrador [EFLAGS](../aprofundando-em-assembly/flags-do-processador.md#system-flags). Quando a TF está ligada cada instrução executada dispara [uma exceção](../aprofundando-em-assembly/interrupcoes-de-software.md) #BP, permitindo assim que o depurador retome o controle após executar uma instrução.

Existe também o conceito de _step over_ que é quando o depurador executa apenas "uma instrução" porém passando todas as instruções da rotina chamada pelo CALL. O que ele faz na prática é definir um _breakpoint_ temporário para a instrução seguinte ao CALL, como na ilustração:

```nasm
    mov rdi, 5     ; Última instrução executada
--> call anything  ; CALL que iremos "passar por cima"
    test rax, rax  ; Instrução onde o breakpoint será definido
```

Se o depurador estiver parado no CALL e executamos um _step over_, o depurador coloca o _breakpoint_ temporário na instrução TEST e então irá executar o processo. Quando o _breakpoint_ na instrução TEST for alcançado ele será removido e o controle será dado para o usuário.

Repare no "defeito" desse mecanismo. O _step over_ só funciona apropriadamente se a instrução seguinte ao CALL realmente for executada, senão o processo continuará a execução normalmente. Experimente  rodar o seguinte código em um depurador:

{% code title="testing.asm" %}
```nasm
bits 64
default rel

SYS_EXIT equ 60

section .text

global _start
_start:
	call oops
	nop
	xor rdi, rdi
	mov rax, SYS_EXIT
	syscall

oops:
	add qword [rsp], 1
	ret
```
{% endcode %}

Compile com:

```shell-session
$ nasm testing.asm -o testing.o -felf64
$ ld testing.o -o testing
```

Ao dar um _step over_ na chamada `call oops` um comportamento inesperado ocorre, o programa irá finalizar sem parar após o retorno da chamada. Isso é demonstrado na imagem abaixo com o depurador GDB:

![Saída do depurador GDB](<../.gitbook/assets/image (11).png>)

### Informações de depuração do executável

Muitos depuradores voltados para desenvolvedores leem informações de depuração à respeito do executável produzidas pelo próprio compilador. O compilador pode, por exemplo, dar informações para que o depurador seja capaz de identificar de qual arquivo e linha do código-fonte uma instrução pertence.

É assim que funcionam os depuradores que exibem o código-fonte (ao invés de apenas as instruções em Assembly) enquanto executam o processo.

No caso do GCC ele armazena essas informações dentro do próprio executável na tabela de símbolos. Já o compilador da Microsoft, usado no Visual Studio, atualmente gera um arquivo `.pdb` contendo todas as informações de depuração.

Vale ressaltar aqui que o GCC (e qualquer outro compilador) **não** armazena o código-fonte do projeto dentro do executável. Ele meramente armazena o endereço do arquivo lá.

É comum também que depuradores apresentem algum erro ao não encontrar o arquivo-fonte indicado no endereço armazenado nas informações de depuração. Isso acontece quando ele tenta apresentar uma linha de código naquele arquivo mas o mesmo não foi encontrado na sua máquina.
