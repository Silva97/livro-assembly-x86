---
description: Entendendo a sintaxe da linguagem Assembly no nasm
---

# Sintaxe

O Assembly da arquitetura x86 tem duas versões diferentes de sintaxe: A sintaxe Intel e a sintaxe AT\&T.\
A sintaxe Intel é a que iremos usar neste livro já que, ao meu ver, ela é mais intuitiva e legível. Também é a sintaxe que o nasm usa, já o GAS suporta as duas porém usando sintaxe AT\&T por padrão. É importante saber ler código das duas sintaxes, mas por enquanto vamos aprender apenas a sintaxe do nasm.

### Case Insensitive

As instruções da linguagem Assembly, bem como também as instruções particulares do nasm, são _case-insensitive_. O que significa que não faz diferença se eu escrevo em caixa-alta, baixa ou mesclando os dois. Veja que cada linha abaixo o nasm irá compilar como a mesma instrução:

```nasm
mov eax, 777
Mov Eax, 777
MOV EAX, 777
mov EAX, 777
MoV EaX, 777
```

### Comentários

No nasm se pode usar o ponto-vírgula `;` para comentários que única linha, equivalente ao `//` em C.\
Comentários de múltiplas linhas podem ser feitos usando a diretiva pré-processada `%comment` para iniciar o comentário e `%endcomment` para finalizá-lo. Exemplo:

```nasm
; Um exemplo
mov eax, 777 ; Outro exemplo

%comment
  Mais
  um
  exemplo
%endcomment
```

### Números

Números literais podem ser escritos em base decimal, hexadecimal, octal e binário. Também é possível escrever constantes numéricas de ponto flutuante no nasm, conforme exemplos:

| Exemplo | Formato         |
| ------- | --------------- |
| 0b0111  | Binário         |
| 0o10    | Octal           |
| 9       | Decimal         |
| 0x0a    | Hexadecimal     |
| 11.0    | Ponto flutuante |

### Strings

Strings podem ser escritas no nasm de três formas diferentes:

| Representação | Explicação                                                       |
| ------------- | ---------------------------------------------------------------- |
| "String"      | String normal                                                    |
| 'String'      | String normal, equivalente a usar "                              |
| \`String\n\`  | String que aceita caracteres de escape no estilo da linguagem C. |

Os dois primeiros são equivalentes e não tem nenhuma diferença para o nasm. O último aceita caracteres de escape no mesmo estilo da linguagem C.

### Formato das instruções

As instruções em Assembly seguem a premissa de especificar uma operação e seus operandos. Na arquitetura x86 uma instrução pode não ter operando algum e chegar até três operandos.

```
operação operando1, operando2, operando3
```

Algumas instruções alteram o valor de um ou mais operandos, que pode ser um endereçamento na memória ou um registrador. Nas instruções que alteram o valor de apenas um operando ele sempre será o operando mais à esquerda. Um exemplo prático é a instrução **mov**:

```nasm
mov eax, 777
```

O `mov` especifica a operação enquanto o `eax` e o `777` são os operandos. Essa instrução altera o valor do operando destino `eax` para `777`. Exemplo de pseudo-código:

```c
eax = 777;
```

{% hint style="danger" %}
Da mesma forma que não é possível fazer `777 = eax`em linguagens de alto nível, também não dá para passar um valor numérico como operando destino para `mov`. Ou seja, isto está errado:\
`mov 777, eax`
{% endhint %}

### Endereçamento

O endereçamento em Assembly x86 é basicamente um cálculo para acessar determinado valor na memória. O resultado deste cálculo é o endereço na memória que o processador irá acessar, seja para ler ou escrever dados no mesmo. Usá-se os colchetes `[]` para denotar um endereçamento. Ao usar colchetes como operando você está basicamente acessando um valor na memória. Por exemplo poderíamos alterar o valor no endereço `0x100` usando a instrução **mov** para o valor contido no registrador `eax`.

```nasm
mov [0x100], eax
```

Como eu já mencionei o valor contido dentro dos colchetes é um cálculo. Vamos aprender mais à respeito quando eu for falar de endereçamento na memória.

{% hint style="danger" %}
Você só pode usar **um** operando na memória por instrução. Então não é possível fazer algo como:\
`mov [0x100], [0x200]`
{% endhint %}

### Tamanho do operando

Quando um dos operandos é um endereçamento na memória você precisa especificar o seu tamanho.\
Ao fazer isso você define o número de bytes que serão lidos ou escritos na memória. A maioria das instruções exigem que o operando destino tenha o mesmo tamanho do operando que irá definir o seu valor, salvo algumas exceções. No nasm existem palavra-chaves (_keywords_) que você pode posicionar logo antes do operando para determinar o seu tamanho.

| Nome  | Nome estendido | Tamanho do operando (em bytes) |
| ----- | -------------- | ------------------------------ |
| byte  |                | 1                              |
| word  |                | 2                              |
| dword | double word    | 4                              |
| qword | quad word      | 8                              |
| tword | ten word       | 10                             |
| oword |                | 16                             |
| yword |                | 32                             |
| zword |                | 64                             |

Exemplo:

```nasm
mov dword [0x100], 777
```

Se você usar um dos operandos como um registrador o nasm irá automaticamente assumir o tamanho do operando como o mesmo tamanho do registrador. Esse é o único caso onde você não é obrigado a especificar o tamanho porém em algumas instruções o nasm não consegue inferir o tamanho do operando.

### Pseudo-instruções

No nasm existem o que são chamadas de "pseudo-instruções", são instruções que não são de fato instruções da arquitetura x86 mas sim instruções que serão interpretadas pelo nasm. Elas são úteis para deixar o código em Assembly mais versátil mas deixando claro que elas não são instruções que serão executadas pelo processador. Exemplo básico é a pseudo-instrução `db` que serve para despejar bytes no correspondente local do arquivo binário de saída. Observe:

```nasm
db 0x41, 0x42, 0x43, 0x44, "String", 0
```

Dá para especificar o byte como um número ou então uma sequência de bytes em formato de string. Essa pseudo-instrução não tem limite de valores separados por vírgula. Veja a saída do exemplo acima no hexdump, um visualizador hexadecimal:

![](<../.gitbook/assets/Captura de tela de 2019-07-16 10-52-25.png>)

### Rótulos

Os rótulos, ou em inglês _labels_, são definições de símbolos usados para identificar determinados endereços da memória no código fonte em Assembly. Podem ser usados de maneira bastante parecida com os rótulos em C. O nome do rótulo serve para pegar o endereço da memória do byte seguinte a posição do rótulo, que pode ser uma instrução ou um byte qualquer produzido por uma pseudo-instrução.\
Para escrever um rótulo basta digitar seu nome seguido de dois-pontos `:`

```
meu_rotulo: instrução/pseudo-instrução
```

Você pode inserir instruções/pseudo-instruções imediatamente após o rótulo ou então em qualquer linha seguinte, não faz diferença no resultado final. Também é possível adicionar um rótulo no final do arquivo, o fazendo apontar para o byte seguinte ao conteúdo do arquivo na memória.\
\
Já vimos um exemplo prático de uso de rótulo na nossa PoC:

```nasm
bits 64

global assembly
assembly:
  mov eax, 777
  ret
```

Repare o rótulo `assembly` na linha 4. Nesse caso o rótulo está sendo usado para denotar o símbolo que aponta para a primeira instrução da nossa função homônima.

### Rótulos locais

Um rótulo local, em inglês _local label_, é basicamente um rótulo que hierarquicamente está abaixo de outro rótulo. Para definir um rótulo local podemos simplesmente adicionar um ponto `.` como primeiro caractere do nosso rótulo. Veja o exemplo:

```nasm
meu_rotulo:
  mov eax, 777
.subrotulo:
  mov ebx, 555
```

Dessa forma o nome completo de `.subrotulo` é na verdade `meu_rotulo.subrotulo`. As instruções que estejam hierarquicamente dentro do rótulo "pai" podem acessar o rótulo local usando de sua nomenclatura com `.` no início do nome ao invés de citar o nome completo. Como no exemplo:

```nasm
meu_rotulo:
  jmp .subrotulo
  mov eax, 777

.subrotulo:
  ret
```

{% hint style="info" %}
Não se preocupe se não entendeu direito, isso aqui é apenas para ver a sintaxe. Vamos aprender mais sobre os rótulos e símbolos depois.
{% endhint %}

### Diretivas

Parecido com as pseudo-instruções, o nasm também oferece as chamadas diretivas. A diferença é que as pseudo-instruções apresentam uma saída em bytes exatamente onde elas são utilizadas, já as diretivas são como comandos para modificar o comportamento do assembler.

Por exemplo a diretiva `bits` que serve para especificar se as instruções seguintes são de 64, 32 ou 16 bits. Podemos observar o uso desta diretiva na nossa PoC. Por padrão o nasm monta as instruções como se fossem de 16 bits.
