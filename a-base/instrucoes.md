---
description: Algumas instruções do ASM x86
---

# Instruções

Já expliquei os conceitos principais por trás da linguagem Assembly da arquitetura x86, agora que já entendemos como a base funciona precisamos nos munir de algumas instruções para poder fazer códigos mais complexos.  
Pensando nisso vou listar aqui algumas instruções e uma explicação bem básica de como utilizá-la.

### Formato de uma instrução

Já expliquei a sintaxe de uma instrução, mas não expliquei o formato em si da instrução no código de máquina. Para simplificar aqui uma instrução pode ter os seguintes operandos:

* Um operando registrador
* Um operando registrador OU operando na memória
* Um valor imediato, que é um operando digitado diretamente

Basicamente são três tipos de operandos: Um registrador, valor na memória e um valor imediato. Um exemplo de cada um para ilustrar sendo mostrado como o segundo operando de MOV:

```text
mov eax, ebx      ; EBX   = Registrador
mov eax, [ebx]    ; [EBX] = Memória
mov eax, 65       ; 65    = Valor imediato
mov eax, "A"      ; "A"   = Valor imediato, mesmo que 65
```

{% hint style="info" %}
Como demonstrado na linha 4, strings podem ser passadas como um operando imediato. O assembler irá converter a string em sua respectiva representação em bytes, só que é necessário ter atenção em relação ao tamanho da string que não pode ser maior do que o operando destino.
{% endhint %}

São três operandos diferentes e cada um deles é opcional, isto é, pode ou não ser utilizado pela instrução. \(opcional para a instrução e não para nós\)  
Repare que somente um dos operandos pode ser um valor na memória ou registrador, enquanto o outro é especificamente um registrador.  
É devido a isto que nasce a limitação que haver apenas um operando na memória, enquanto que o uso de dois registradores é permitido.

### Notação

{% hint style="info" %}
Irei utilizar uma explicação simplificada aqui que irá deixar muita informação importante de fora. Depois irei especificar as instruções com mais detalhes.
{% endhint %}

As seguintes nomenclaturas serão utilizadas:

| Nomenclatura | Significado |
| :--- | :--- |
| reg | Um operando registrador |
| r/m | Um operando registrador ou na memória |
| imm | Um operando imediato |
| addr | Denota um endereço, geralmente se usa um rótulo. Na prática é um valor imediato assim como o operando imediato. |

Em alguns casos eu posso colocar um número junto a esta nomenclatura para especificar o tamanho do operando em bits. Por exemplo `r/m16` indica um operando registrador/memória de 16 bits.

Em cada instrução irei apresentar a notação demonstrando cada combinação diferente de operandos que é possível utilizar.  
Lembrando que o **operando destino** é o mais a esquerda, enquanto que o **operando fonte** é o operando mais a direita.

{% hint style="info" %}
Cada nome de uma instrução em Assembly é um mnemônico, que é basicamente uma abreviatura feita para fácil memorização.  
Pensando nisto, leia cada instrução com seu nome extenso equivalente para lembrar o que ela faz.  
No título de cada instrução irei deixar após um "\|" o nome extenso da instrução para facilitar nesta tarefa.
{% endhint %}

### MOV \| Move

```text
mov reg, r/m
mov reg, imm
mov r/m, reg
mov r/m, imm
```

Copia o valor do operando fonte para o operando destino.

{% code title="pseudo.c" %}
```c
destiny = source;
```
{% endcode %}

### ADD

```text
add reg, r/m
add reg, imm
add r/m, reg
add r/m, imm
```

Soma o valor do operando destino com o valor do operando fonte.

{% code title="pseudo.c" %}
```c
destiny = destiny + source;
```
{% endcode %}

### SUB \| Subtract

```text
sub reg, r/m
sub reg, imm
sub r/m, reg
sub r/m, imm
```

Subtrai o valor do operando destino com o valor do operando fonte.

{% code title="pseudo.c" %}
```c
destiny = destiny - source;
```
{% endcode %}

### INC \| Increment

```text
inc r/m
```

Incrementa o valor do operando destino em 1.

{% code title="pseudo.c" %}
```c
destiny++;
```
{% endcode %}

### DEC \| Decrement

```text
dec r/m
```

Decrementa o valor do operando destino em 1.

{% code title="pseudo.c" %}
```c
destiny--;
```
{% endcode %}

### MUL \| Multiplicate

```text
mul r/m
```

Multiplica uma parte do mapeamento de RAX pelo operando passado.  
Com base no tamanho do operando uma parte diferente de RAX será multiplicada e o resultado armazenado em um registrador diferente.

| Operando 1 | Operando 2 | Destino |
| :--- | :--- | :--- |
| AL | r/m8 | AX |
| AX | r/m16 | DX:AX |
| EAX | r/m32 | EDX:EAX |
| RAX | r/m64 | RDX:RAX |

No caso por exemplo de DX:AX, os registradores de 16 bits são usados em conjunto para representar o valor de 32 bits. Onde DX armazena os 2 bytes mais significativos do valor, e AX os 2 bytes menos significativos.

{% code title="pseudo.c" %}
```c
// Se operando de 8 bits
AX = AL * operand;

// Se operando de 16 bits
aux = AX * operand;
DX  = (aux & 0xffff0000) >> 16;
AX  = aux & 0x0000ffff;
```
{% endcode %}

### DIV \| Divide

```text
div r/m
```

Seguindo uma premissa inversa de MUL, faz a divisão de um valor pelo operando passado e armazena o quociente e a sobra desta divisão.

| Operando 1 | Operando 2 | Destino quociente | Destino sobra |
| :--- | :--- | :--- | :--- |
| AX | r/m8 | AL | AH |
| DX:AX | r/m16 | AX | DX |
| EDX:EAX | r/m32 | EAX | EDX |
| RDX:RAX | r/m64 | RAX | RDX |

{% code title="pseudo.c" %}
```c
// Se operando de 8 bits
AL = AX / operand;
AH = AX % operand;
```
{% endcode %}

### LEA \| Load Effective Address

```text
lea reg, mem
```

Calcula o endereço efetivo do operando fonte e armazena o resultado no endereço destino.

{% code title="pseudo.c" %}
```c
destiny = address_of(source);
```
{% endcode %}

### AND

```text
and reg, r/m
and reg, imm
and r/m, reg
and r/m, imm
```

Faz uma operação _E_ bit a bit nos operandos e armazena o resultado no operando destino.

{% code title="pseudo.c" %}
```c
destiny = destiny & source;
```
{% endcode %}

### OR

```text
or reg, r/m
or reg, imm
or r/m, reg
or r/m, imm
```

Faz uma operação _OU_ bit a bit nos operandos e armazena o resultado no operando destino.

{% code title="pseudo.c" %}
```c
destiny = destiny | source;
```
{% endcode %}

### XOR \| Exclusive OR

```text
xor reg, r/m
xor reg, imm
xor r/m, reg
xor r/m, imm
```

Faz uma operação _OU Exclusivo_ bit a bit nos operandos e armazena o resultado no operando destino.

{% code title="pseudo.c" %}
```c
destiny = destiny ^ source;
```
{% endcode %}

### XCHG \| Exchange

```text
xchg reg, r/m
xchg r/m, reg
```

O operando destino recebe o valor do operando fonte, e o operando fonte recebe o valor anterior do operando destino.  
Fazendo assim uma troca nos valores dos mesmos.

```c
auxiliary = destiny;
destiny   = source;
source    = auxiliary;
```

### XADD \| Exchange and Add

```text
xadd r/m, reg
```

O operando fonte recebe o valor do operando destino e, em seguida, o operando destino é somado com o valor anterior do operando fonte.  
Basicamente preserva o valor do destino ao mesmo tempo que faz um ADD nele.

{% code title="pseudo.c" %}
```c
auxiliary = source;
source    = destiny;
destiny   = destiny + auxiliary;
```
{% endcode %}

### SHL \| Shift Left

```text
shl r/m
shl r/m, imm
shl r/m, CL
```

Faz o deslocamento de bits do operando destino para a esquerda com base no número especificado no operando fonte.  
Se o operando fonte não é especificado, então faz o _shift left_ apenas 1 vez.

{% code title="pseudo.c" %}
```c
destiny = destiny << 1;       // Se: shl r/m
destiny = destiny << source; //  Nos outros casos
```
{% endcode %}

### SHR \| Shift Right

```text
shr r/m
shr r/m, imm
shr r/m, CL
```

Mesmo caso que SHL, porém faz o deslocamento de bits para a direita.

{% code title="pseudo.c" %}
```c
destiny = destiny >> 1;       // Se: shr r/m
destiny = destiny >> source; //  Nos outros casos
```
{% endcode %}

### CMP \| Compare

```text
cmp r/m, imm
cmp r/m, reg
cmp reg, r/m
```

Compara o valor dos dois operandos e define RFLAGS de acordo.

{% code title="pseudo.c" %}
```c
RFLAGS = compare(operand1, operand2);
```
{% endcode %}

### SETcc \| Set byte if Condition

```text
SETcc r/m8
```

Define o valor do operando de 8 bits para 1 ou 0 dependendo se a condição for atendida \(1\) ou não \(0\).  
Assim como no caso dos _jumps_ condicionais, o 'cc' aqui denota uma sigla para uma condição. Cuja elas podem ser as mesmas utilizadas nos _jumps_. Exemplo:

```text
sete al
; Se RFLAGS indica um valor igual, AL = 1. Se não AL = 0
```

{% code title="pseudo.c" %}
```c
if(verify_rflags(condition) == true) destiny = 1;
else                                 destiny = 0;
```
{% endcode %}

### CMOVcc \| Conditional Move

```text
CMOVcc reg, r/m
```

Basicamente uma instrução MOV condicional, só irá definir o valor do operando destino caso a condição seja atendida.

{% code title="pseudo.c" %}
```c
if(verify_rflags(condition) == true) destiny = source;
```
{% endcode %}

### NEG \| Negate

```text
neg r/m
```

Inverte o sinal do valor numérico do operando.

{% code title="pseudo.c" %}
```c
destiny = -destiny;
```
{% endcode %}

### NOT

```text
not r/m
```

Faz uma operação _NÃO_ bit a bit no operando.

{% code title="pseudo.c" %}
```c
destiny = ~destiny;
```
{% endcode %}

### MOVSB/MOVSW/MOVSD/MOVSQ \| Move String

```text
movsb  ; byte        (1 byte)
movsw  ; word        (2 bytes)
movsd  ; double word (4 bytes)
movsq  ; quad word   (8 bytes)
```

Copia um valor do tamanho de um **b**yte, **w**ord, **d**ouble word ou **q**uad word a partir do endereço apontado por RSI \(_Source Index_\) para o endereço apontado por RDI. \(_Destiny Index_\).  
Depois disso incrementa o valor dos dois registradores com base no tamanho do dado que foi movido.

{% code title="pseudo.c" %}
```c
// Se MOVSW
word [RDI] = word [RSI];
RDI        = RDI + 2;
RSI        = RSI + 2;
```
{% endcode %}

### CMPSB/CMPSW/CMPSD/CMPSQ \| Compare String

```text
cmpsb  ; byte        (1 byte)
cmpsw  ; word        (2 bytes)
cmpsd  ; double word (4 bytes)
cmpsq  ; quad word   (8 bytes)
```

Compara os valores na memória apontados por RDI e RSI, e depois incrementa os registradores com base no tamanho do dado.

{% code title="pseudo.c" %}
```c
// CMPSW
RFLAGS = compare(word [RDI], word [RSI]);
RDI    = RDI + 2;
RSI    = RSI + 2;
```
{% endcode %}

### LODSB/LODSW/LODSD/LODSQ \| Load String

```text
lodsb  ; byte        (1 byte)
lodsw  ; word        (2 bytes)
lodsd  ; double word (4 bytes)
lodsq  ; quad word   (8 bytes)
```

Copia o valor na memória apontado por RSI para uma parte do mapeamento de RAX equivalente ao tamanho do dado, e depois incrementa RSI de acordo.

{% code title="pseudo.c" %}
```c
// LODSW
AX  = word [RSI];
RSI = RSI + 2;
```
{% endcode %}

### SCASB/SCASW/SCASD/SCASQ \| Scan String

```text
scasb  ; byte        (1 byte)
scasw  ; word        (2 bytes)
scasd  ; double word (4 bytes)
scasq  ; quad word   (8 bytes)
```

Compara o valor em uma parte mapeada de RAX com o valor na memória apontado por RDI, e depois incrementa RDI de acordo.

{% code title="pseudo.c" %}
```c
// SCASW
RFLAGS = compare(AX, word [RDI]);
RDI    = RDI + 2;
```
{% endcode %}

### STOSB/STOSW/STOSD/STODQ \| Store String

```text
stosb  ; byte        (1 byte)
stosw  ; word        (2 bytes)
stosd  ; double word (4 bytes)
stosq  ; quad word   (8 bytes)
```

Copia o valor de uma parte mapeada de RAX e armazena na memória apontada por RDI, depois incrementa RDI de acordo.

{% code title="pseudo.c" %}
```c
// STOSW
word [RDI] = AX;
RDI        = RDI + 2;
```
{% endcode %}

### LOOP/LOOPE/LOOPNE

{% code title="pseudo.c" %}
```c
loop   addr8
loope  addr8
loopne addr8
```
{% endcode %}

Essas instruções são utilizadas para gerar procedimentos de laço \(_loop_\) usando o registrador RCX como contador.  
Elas primeiro decrementam o valor de RCX e comparam o mesmo com o valor zero. Se RCX for diferente de zero, a instrução faz um salto para o endereço passado com operando, senão o fluxo de código continua normalmente.  
No caso de `loope` e `loopne`, os sufixos indicam a condição de **igual** e **não igual** respectivamente.  
Ou seja, além da comparação do valor de RCX elas também verificam o valor de RFLAGS como uma condição extra.

{% code title="pseudo.c" %}
```c
// loop
RCX = RCX - 1;
if(RCX != 0) goto operand;

// loope
RCX = RCX - 1;
if(RCX != 0 && verify_rflags(EQUAL) == true) goto operand;

// loopne
RCX = RCX - 1;
if(RCX != 0 && verify_rflags(EQUAL) == false) goto operand;
```
{% endcode %}

### NOP \| No Operation

```text
nop
```

Não faz nenhuma operação... Sério, não faz nada.

{% code title="pseudo.c" %}
```c
EAX = EAX;
```
{% endcode %}

