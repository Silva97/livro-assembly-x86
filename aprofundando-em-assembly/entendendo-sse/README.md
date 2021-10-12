---
description: Aprendendo sobre SIMD, SSE e registradores XMM.
---

# Entendendo SSE

Na computação existe um conceito de instrução chamado SIMD (_Single Instruction, Multiple Data_) que é basicamente uma instrução que processa múltiplos dados de uma única vez. Todas as instruções que vimos até agora processavam meramente um dado por vez, porém instruções SIMD podem processar diversos dados paralelamente. O principal objetivo das instruções SIMD é ganhar performance se aproveitando dos múltiplos núcleos do processador, a maioria das instruções SIMD foram implementadas com o intuito de otimizar cálculos comuns em áreas como processamento gráfico, inteligência artificial, criptografia, matemática etc.

A Intel criou a primeira versão do SSE (_streaming SIMD extensions_) ainda no IA-32 com o Pentium III, e de lá para cá já ganhou diversas novas versões que estendem a tecnologia adicionando novas instruções. Atualmente nos processadores mais modernos há as seguintes extensões: SSE, SSE2, SSE3, SSSE3 e SSE4.

{% hint style="info" %}
Processadores da arquitetura x86 têm diversas tecnologias SIMD, a primeira delas foi o MMX nos processadores Intel antes mesmo do SSE. Além de haver diversas outras como AVX,  AVX-512, FMA, 3DNow! (da AMD) etc.

Na arquitetura x86 existem literalmente milhares de instruções SIMD. Esteja ciente que esse tópico está longe de cobrir todo o assunto e serve meramente como conteúdo introdutório.
{% endhint %}

## Registradores XMM

A tecnologia SSE adiciona novos registradores independentes de 128 bits de tamanho cada. Em todos os modos de processamento são adicionados oito novos registradores **XMM0** até **XMM7**, e em 64-bit também há mais oito registradores **XMM8 **até **XMM15** que podem ser acessados usando o [prefixo REX](../prefixos.md#rex). Isso dá um total de 16 registradores em 64-bit e 8 registradores nos outros modos de processamento.

Esses registradores podem armazenar vários dados diferentes de mesmo tipo/tamanho, conforme demonstra tabela abaixo:

![Intel Developer's Manuals | 4.6.2 128-Bit Packed SIMD Data Types](<../../.gitbook/assets/image (9).png>)

Esses são os tipos empacotados (_packed_), onde em um único registrador há vários valores de um mesmo tipo. Existem instruções SIMD específicas que executam operações _packed_ onde elas trabalham com os vários dados armazenados no registrador ao mesmo tempo. Em contraste existem também as operações escalares (_scalar_) que operam com um único dado (_unpacked_) no registrador, onde esse dado estaria armazenado na parte menos significativa do registrador.

{% hint style="info" %}
Na convenção de chamada para x86-64 da linguagem C os primeiros argumentos _float/double_ passados para uma função vão nos registradores XMM0, XMM1 etc. como valores escalares. E o retorno do tipo _float/double_ fica no registrador XMM0 também como um valor escalar.

Na lista de instruções haverá códigos de exemplo disso.
{% endhint %}

## Entendendo as instruções SSE

As instruções adicionadas pela tecnologia SSE podem ser divididas em quatro grupos:

* Instruções _packed_ e _scalar_ que lidam com números _float_.
* Instruções SIMD com inteiros de 64 bits.
* Instruções de gerenciamento de estado.
* Instruções de controle de cache e _prefetch_.

A tabela abaixo lista a nomenclatura que irei utilizar para descrever as instruções SSE.

Para facilitar o entendimento irei usar o termo _float_ para se referir aos números de ponto flutuante de precisão única, ou seja, 32 bits de tamanho e 23 bits de precisão. Já o termo _double_ será utilizado para se referir aos números de ponto flutuante de dupla precisão, ou seja, de 64 bits de tamanho e 52 bits de precisão. Esses são os mesmos nomes usados como tipos na linguagem C.

| Nomenclatura | Descrição                                                                                                                              |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| xmm(n)       | Indica qualquer um dos registradores XMM.                                                                                              |
| float(n)     | Indica N números_ floats _em sequência na memória RAM. Exemplo: **float(4)** seriam 4 números _float_ totalizando 128 bits de tamanho. |
| double(n)    | Indica N números _double_ na memória RAM. Exemplo: **double(2)** que totaliza 128 bits.                                                |
| ubyte(n)     | Indica N bytes não-sinalizados na memória RAM. Exemplo: **ubyte(16)** que totaliza 128 bits.                                           |
| byte(n)      | Indica N bytes sinalizados na memória RAM.                                                                                             |
| uword(n)     | Indica N _words_ (2 bytes) não-sinalizados na memória RAM. Exemplo: **uword(8)** que totaliza 128 bits.                                |
| word(n)      | Indica N _words_ sinalizadas na memória RAM.                                                                                           |
| dword(n)     | Indica N _double words_ (4 bytes) na memória RAM.                                                                                      |
| qword(n)     | Indica N _quadwords _(8 bytes) na memória RAM.                                                                                         |
| reg32/64     | [Registrador de propósito geral](../../a-base/registradores-gerais.md) de 32 ou 64 bits.                                               |
| imm8         | Operando imediato de 8 bits de tamanho.                                                                                                |

As instruções SSE terminam com um sufixo de duas letras onde a penúltima indica se ela lida com dados _packed_ ou _scalar_, e a última letra indica o tipo do dado sendo manipulado. Por exemplo a instrução MOVA**PS** onde o **P** indica que a instrução manipula dados _packed_, enquanto o **S** indica o tipo do dado como _**s**ingle-precision floating-point_, ou seja, _float_ de 32 bits de tamanho.

Já o **D** de MOVAP**D** indica que a instrução lida com valores do tipo _double-precision floating-point_ (64 bits). Eis a lista de sufixos e seus respectivos tipos:

| Sufixo | Tipo                                                                                                                                                                      |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| S      | _Single-precision float_. Equivalente ao tipo **float** em C.                                                                                                             |
| D      | <p><em>Double-precision float</em>. Equivalente ao tipo <strong>double</strong> em C.</p><p>Ou inteiro <em>doubleword</em> (4 bytes) que seria um inteiro de 32 bits.</p> |
| B      | Inteiro de um byte (8 bits).                                                                                                                                              |
| W      | Inteiro word (2 bytes \| 16 bits).                                                                                                                                        |
| Q      | Inteiro _quadword_ (8 bytes \| 64 bits).                                                                                                                                  |

{% hint style="danger" %}
Todas as instruções SSE que lidam com valores na memória **exigem** que o valor esteja em um endereço alinhado em 16 bytes, exceto as instruções que explicitamente dizem lidar com dados desalinhados (_unaligned_).

Caso uma instrução SSE seja executada com um dado desalinhado uma exceção #GP será disparada.
{% endhint %}
