---
description: Segmentação da memória RAM.
---

# Registradores de segmento

Na arquitetura x86 o acesso a memória RAM é comumente dividido em segmentos. Um segmento de memória nada mais é que um pedaço da memória RAM que o programador usa dando algum sentido a ele. Por exemplo, podemos usar um segmento só para armazenar variáveis. E usar outro para armazenar o código executado pelo processador.

{% hint style="warning" %}
Rodando sob um sistema operacional a segmentação da memória é totalmente controlada pelo _kernel_. Ou seja, não tente fazer o que você não tem permissão. 😉 
{% endhint %}

### Barramento de endereço

O barramento de endereço \(_address bus_\) é um _socket_ do processador que serve para se comunicar com a memória principal \(memória RAM\), ele indica o endereço físico na memória principal de onde o processador quer ler ou escrever dados. Basicamente a largura desse barramento indica quanta memória o processador consegue endereçar já que ele indica o endereço físico da memória que se deseja acessar.

Em IA-16 o barramento tem o tamanho padrão de 20 bits. Calculando  $$2^{20}$$ temos o número de bytes endereçáveis que são exatamente 1 MiB de memória que pode ser endereçada. É da largura do barramento de endereço que surge a limitação de tamanho da memória RAM.

Em IA-32 e x86-64 o barramento de endereço tem a largura de 32 e 48 bits respectivamente.

### Segmentação em IA-16

Em IA-16 a segmentação é bem simplista e o código trabalha basicamente com 4 segmentos simultaneamente. Esses segmentos são definidos simplesmente alterando o registrador de segmento equivalente, cujo eles são:

| Registrador | Nome |
| :--- | :--- |
| CS | Code Segment / Segmento de código |
| DS | Data Segment / Segmento de dado |
| ES | Extra Segment / Segmento extra |
| SS | Stack Segment / Segmento da pilha |

Cada um desses registradores tem 16 bits de tamanho.

Quando acessamos um endereço na memória estamos usando um endereço lógico que é a junção de um segmento \(_segment_\) e um deslocamento \(_offset_\), seguindo o formato:  
`segment:offset`

{% hint style="info" %}
O tamanho do valor de _offset_ é o mesmo tamanho do registrador IP/EIP/RIP.
{% endhint %}

Veja por exemplo a instrução:

```text
mov [0x100], ax
```

O endereçamento definido pelos colchetes é na verdade o _offset_ que, juntamente com o registrador DS, se obtém o endereço físico. Ou seja o endereço lógico é `DS:0x100`.

{% hint style="info" %}
O segmento padrão \(nesse caso DS\) usado para acessar o endereço depende de qual registrador e instrução está sendo utilizado. No tópico [Atributos](atributos.md#segment) isso será explicado.
{% endhint %}

Podemos especificar um segmento diferente com a seguinte sintaxe do NASM:

```text
; O nome deste recurso é "segment override"
; Ou em PT-BR: Substituição do segmento

mov [es:0x100], ax

; OU alternativamente:

es mov [0x100], ax
```

A conversão de endereço lógico para endereço físico é feita pelo processador com um cálculo simples:

```text
endereço_físico = (segmento << 4) + deslocamento
```

O operador `<<` denota um deslocamento de bits para a esquerda, uma operação _shift left_.

### Segmentação em IA-32

Além dos registradores de segmento do IA-16, em IA-32 se ganha mais dois registradores de segmento:  `FS` e `GS`.

Diferente dos registradores gerais, os registradores de segmento não são expandidos. Permanecem com o tamanho de 16 bits.

Em _protected mode_ os registradores de segmento não são usados para gerar um endereço lógico junto com o _offset_, ao invés disso, serve de seletor identificando o segmento por um índice em uma tabela que lista os segmentos.

### Segmentação em x86-64

Em x86-64 não é mais usado esse esquema de segmentação de memória. CS, DS, ES e SS são tratados como se o endereço base fosse zero independentemente do valor nesses registradores.

Já os registradores FS e GS são exceções e ainda podem ser usados pelo sistema operacional para endereçamento de estruturas especiais na memória. Como por exemplo no Linux, em x86-64, FS é usado para apontar para a [Thread Local Storage](../programando-junto-com-c/variaveis-em-c.md#variaveis-_thread_local).

