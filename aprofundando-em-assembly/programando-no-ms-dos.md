---
description: Conhecendo o ambiente do MS-DOS.
---

# Programando no MS-DOS

O clássico MS-DOS, antigo sistema operacional de 16 bits da Microsoft, foi muito utilizado e até hoje existem projetos relacionados a esse sistema. Existe por exemplo o [FreeDOS](https://www.freedos.org) que é um sistema operacional de código aberto e que é compatível com o MS-DOS.

A famosa "telinha preta" do Windows, o prompt de comando, muitas vezes é erroneamente chamado de MS-DOS devido aos dois usarem o mesmo shellscript chamado de Batch. Isso fazia com que comandos rodados no MS-DOS fossem quase totalmente compatíveis na linha de comando do Windows.

Mas o prompt de comandos do Windows **não** é o MS-DOS. Esse é apenas o Terminal do sistema operacional Windows e que usa uma versão mais avançada do mesmo shellscript que rodava no MS-DOS.

### Real mode

O MS-DOS era um sistema operacional que rodava em modo de processamento _real mode_, o famoso modo de 16-bit que é compatível com o 8086 original.

### Text mode

Existem modos diferentes de se usar a saída de vídeo, isto é, o monitor do computador. Dentre os vários modos que o monitor suporta, existe a divisão entre modo de texto (_text mode_) e modo de vídeo (_video mode_).

O modo de vídeo é este modo que o seu sistema operacional está rodando agora. Nele o software define informações de cor para **cada pixel** da tela, formando assim imagens desde mais simples (formas opacas) até as mais complexas (imagens renderizadas tridimensionalmente). Todas essas imagens que você vê são geradas pixel a pixel para serem apresentadas pelo monitor.

Já o MS-DOS rodava em modo de texto, cujo este modo é bem mais simples. Ao invés de você definir cada pixel que o monitor apresenta, você define unicamente informações de caracteres. Imagine por exemplo que seu monitor seja dividido em grade formando 80x25 quadrados na tela. Ou seja, 80 colunas e 25 linhas. Ao invés de definir cada pixel você apenas definia qual caractere seria apresentado naquele quadrado e um atributo para esse caractere.

### Executáveis .COM

O formato de executável mais básico que o MS-DOS suportava era os de extensão **.com** que era um _raw binary_. Esse termo é usado para se referir a um "binário puro", isto é, um arquivo binário que não tem qualquer tipo de formatação especial.

Uma comparação com arquivos de texto seria você comparar um código fonte em C com um arquivo de texto "normal". O código fonte em C também é um arquivo de texto, porém ele tem formatações especiais que seguem a sintaxe da linguagem de programação. Enquanto o arquivo de texto "normal" é apenas texto, sem seguir qualquer regra de formatação.

No caso do _raw binary_ é a mesma coisa, informação binária sem qualquer regra de formatação especial.\
Este executável do MS-DOS tinha como "_entry point_" logo o primeiro byte do arquivo. Como eu já disse, não tinha qualquer regra especial nele então você poderia organizá-lo da maneira que quisesse manualmente.

### Execução do .COM

O processo que o MS-DOS fazia para executar esse tipo de executável era tão simples quanto possível. Seguindo o fluxo:

* Recebe um comando na linha de comando.
* Coloca o tamanho em bytes dos argumentos passados pela linha de comando no _offset_ 0x80 do segmento do executável.
* Coloca os argumentos da linha de comando no _offset_ 0x81 como texto puro, sem qualquer formatação.
* Carrega todo o **.COM** no _offset_ 0x100
* Define os registradores DS, SS e ES para o segmento onde o executável foi carregado.
* Faz um `call` no endereço onde o executável foi carregado.

Perceba que a chamada do executável nada mais é que um `call`, por isso esses executáveis finalizavam simplesmente executando um `ret`. Mais simples impossível, né?

### ORG | Origin

```
org endereço_inicial
```

A essa altura você já deve ter reparado que o NASM calcula o endereço dos rótulos sozinho sem precisar da nossa ajuda, né? Então, mas ele faz isso considerando que o primeiro byte do nosso arquivo binário esteja especificamente no _offset_ **0**. Ou seja, ele começa a contar do zero em diante. No caso de um executável **.COM** ele é carregado no _offset_ 0x100 e não em 0, então o cálculo vai dar errado.

Mas o NASM contém a diretiva `org` que serve para dizer para o NASM a partir de qual endereço ele deve calcular o endereço dos rótulos, ou seja, o endereço de **origem** do nosso binário. Veja o exemplo:

```nasm
bits 16
org 0x100

msg: db "abc"

codigo:
  mov ax, 77
  ret
```

O rótulo `codigo` ao invés de ter o endereço calculado como 0x0003 como normalmente teria, terá o endereço 0x0103 devido ao uso da diretiva `org` na segunda linha.

### Hello World no MS-DOS

Um pequeno exemplo de "Hello World" (ou "Hi") para o MS-DOS:

{% code title="hello.asm" %}
```nasm
bits 16
org 0x100

mov ah, 0x0E
mov al, 'H'
int 0x10

mov al, 'i'
int 0x10

ret
```
{% endcode %}

Experimente compilar como um _raw binary_  com extensão `.com` e depois executar no Dosbox (ou FreeDOS ou qualquer projeto semelhante).

```
$ nasm hello.asm -o hello.com
```

{% hint style="info" %}
A instrução **INT** e o que está acontecendo aí será explicado nos dois tópicos posteriores a esse.
{% endhint %}
