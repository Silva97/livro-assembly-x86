---
description: Aprendendo sobre as instruções intrínsecas na arquitetura x86-64
---

# Instruções intrínsecas

As instruções intrínsecas é um recurso originalmente fornecido pelo compilador Intel C/C++ mas que também é implementado pelo GCC. Se tratam basicamente de tipos especiais e funções que são expandidas _inline_ para alguma instrução do processador, ou seja, é basicamente uma alternativa mais prática e legível do que usar _inline_ Assembly para tudo.

Usando instruções intrínsecas é possível obter o mesmo resultado de usar _inline_ Assembly com a diferença de ter a sintaxe amigável de uma chamada de função.

Para usar instruções intrínsecas é necessário incluir o _header_ **\<immintrin.h>** onde ele declara as funções e os tipos.

{% hint style="info" %}
Para entender apropriadamente as operações e tipos indicados aqui, sugiro que já tenha lido o tópico sobre [SSE](../aprofundando-em-assembly/entendendo-sse/).
{% endhint %}

## Tipos de dados

Os tipos de dados na tabela abaixo servem para indicar como os valores usados na instrução intrínseca serão armazenados.

| Tipo      | Descrição                                                                                                                                                          |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `__m64`   | Tipo usado para representar o conteúdo de um registrador MMX. Pode armazenar 8 valores 8-bit, 4 valores de 16-bit, 2 valores de 32-bit ou 1 valor de 64-bit.       |
| `__m128`  | Representa o conteúdo de um [registrador SSE](../aprofundando-em-assembly/entendendo-sse/#registradores-xmm). Pode armazenar 4 valores _floating-point_ de 32-bit. |
| `__m128d` | Também um registrador SSE porém armazenando 2 _floating-point_ de 64-bit.                                                                                          |
| `__m128i` | Registrador SSE que pode armazenar 16 valores inteiros de 8-bit, 8 valores inteiros de 16-bit, 4 valores inteiros de 32-bit ou 2 valores inteiros de 64-bit.       |
| `__m256`  | Representa o conteúdo de um registrador YMM usado pela tecnologia AVX. Pode armazenar 8 valores _floating-point_ de 32-bit.                                        |
| `__m256d` | Registrador YMM que pode armazenar 4 _floating-point_ de 64-bit.                                                                                                   |
| `__m256i` | Registrador YMM que pode armazenar 32 valores inteiros de 8-bit, 16 valores inteiros de 16-bit, 8 valores inteiros de 32-bit ou 4 valores inteiros de 64-bit.      |
| `__m512`  | Representa o conteúdo de um registrador ZMM usado pela tecnologia AVX-512. Pode armazenar 16 valores _floating-point_ de 32-bit.                                   |
| `__m512d` | Registrador ZMM que pode armazenar 8 valores _floating-point_ de 64-bit.                                                                                           |
| `__m512i` | Registrador ZMM que pode armazenar 64 valores inteiros de 8-bit, 32 inteiros de 16-bit, 16 inteiros de 32-bit ou 8 inteiros de 64-bit.                             |

## Nomenclatura

A maioria das instruções intrínsecas (SIMD) seguem a seguinte convenção de notação:

```
_mm_<operação>_<sufixo>
```

Onde **\<operação>** é a operação que será executada com os dados. O <**sufixo>** indica o tipo de dado na operação. A primeira ou as duas primeiras letras do sufixo indicam se o dado é _packed_ (**p**), _extended packed_ (**ep**) ou escalar (**s**). Os demais caracteres do sufixo indicam o tipo de dado, como mostra a tabela abaixo:

| Sufixo | Tipo                                                     |
| ------ | -------------------------------------------------------- |
| `s`    | _single-precision floating-point_ (**float** de 32-bit)  |
| `d`    | _double-precision floating-point_ (**double** de 64-bit) |
| `i128` | Inteiro sinalizado de 128-bit.                           |
| `i64`  | Inteiro sinalizado de 64-bit.                            |
| `u64`  | Inteiro não-sinalizado de 64-bit.                        |
| `i32`  | Inteiro sinalizado de 32-bit.                            |
| `u32`  | Inteiro não-sinalizado de 32-bit.                        |
| `i16`  | Inteiro sinalizado de 16-bit.                            |
| `u16`  | Inteiro não-sinalizado de 16-bit.                        |
| `i8`   | Inteiro sinalizado de 8-bit.                             |
| `u8`   | Inteiro não-sinalizado de 8-bit.                         |

Exemplo:

```c
double array[] = { 1.0, 2.0 };
__m128d data = _mm_load_pd(array);
```

## Instruções

Abaixo irei listar apenas algumas instruções intrínsecas, em sua maioria relacionadas à tecnologia SSE. Para ver a lista completa sugiro que consulte a referência oficial da Intel no link abaixo:

* [https://software.intel.com/sites/landingpage/IntrinsicsGuide/](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)

{% hint style="info" %}
Algumas instruções intrínsecas não são compiladas para uma só instrução mas sim uma **sequência** de várias delas.
{% endhint %}

### Operações load, store e extract

| Tecnologia | Protótipo                                                     | Instrução                  |
| ---------- | ------------------------------------------------------------- | -------------------------- |
| SSE2       | `__m128d _mm_load_pd (double const* mem_addr)`                | `movapd xmm, m128`         |
| SSE        | `__m128 _mm_load_ps (float const* mem_addr)`                  | `movaps xmm, m128`         |
| SSE2       | `__m128d _mm_load_sd (double const* mem_addr)`                | `movsd xmm, m64`           |
| SSE2       | `__m128i _mm_load_si128 (__m128i const* mem_addr)`            | `movdqa xmm, m128`         |
| SSE        | `__m128 _mm_load_ss (float const* mem_addr)`                  | `movss xmm, m32`           |
| SSE        | `__m128i _mm_loadu_si16 (void const* mem_addr)`               | **Sequência**              |
| SSE2       | `__m128i _mm_loadu_si32 (void const* mem_addr)`               | `movd xmm, m32`            |
| SSE        | `__m128i _mm_loadu_si64 (void const* mem_addr)`               | `movq xmm, m64`            |
| SSE2       | `__m128i _mm_loadu_si128 (__m128i const* mem_addr)`           | `movdqu xmm, m128`         |
| SSE2       | `void _mm_store_pd (double* mem_addr, __m128d a)`             | `movapd m128, xmm`         |
| SSE        | `void _mm_store_ps (float* mem_addr, __m128 a)`               | `movaps m128, xmm`         |
| SSE2       | `void _mm_store_sd (double* mem_addr, __m128d a)`             | `movsd m64, xmm`           |
| SSE2       | `void _mm_store_si128 (__m128i* mem_addr, __m128i a)`         | `movdqa m128, xmm`         |
| SSE        | `void _mm_store_ss (float* mem_addr, __m128 a)`               | `movss m32, xmm`           |
| SSE2       | `int _mm_extract_epi16 (__m128i a, int imm8)`                 | `pextrw r32, xmm, imm8`    |
| SSE4.1     | `int _mm_extract_epi32 (__m128i a, const int imm8)`           | `pextrd r32, xmm, imm8`    |
| SSE4.1     | `long long int _mm_extract_epi64 (__m128i a, const int imm8)` | `pextrq r64, xmm, imm8`    |
| SSE4.1     | `int _mm_extract_epi8 (__m128i a, const int imm8)`            | `pextrb r32, xmm, imm8`    |
| SSE        | `int _mm_extract_pi16 (__m64 a, int imm8)`                    | `pextrw r32, mm, imm8`     |
| SSE4.1     | `int _mm_extract_ps (__m128 a, const int imm8)`               | `extractps r32, xmm, imm8` |
| SSE        | `int _m_pextrw (__m64 a, int imm8)`                           | `pextrw r32, mm, imm8`     |

As operações **load** carregam um valor da memória para um registrador, o conteúdo que deve estar na memória apontada pelo argumento tem que estar de acordo com o tipo da instrução identificado pelo sufixo.

Operações **store** leem um ou mais dados do registrador e escrevem os mesmos no endereço passado como primeiro argumento.

Já a operação **extract** obtém um valor de uma parte do registrador identificado pelo valor imediato passado como segundo argumento. Esse valor é o índice do campo do registrador contando da direita para a esquerda começando em zero.

Exemplos:

{% tabs %}
{% tab title="load_store.c" %}
```c
#include <stdio.h>
#include <immintrin.h>

int main(void)
{
  float number;
  float array[] = {1.0f, 2.0f, 3.0f, 4.0f};
  __m128 data = _mm_load_ps(array);

  _mm_store_ss(&number, data);
  printf("%f\n", number);
  return 0;
}
```
{% endtab %}

{% tab title="load_extract.c" %}
```c
#include <stdio.h>
#include <immintrin.h>

int main(void)
{
  int array[] = {111, 222, 333, 444};
  __m128i data = _mm_load_si128((__m128i *)array);

  int number = _mm_extract_epi32(data, 2);
  printf("%d\n", number);
  return 0;
}
```
{% endtab %}
{% endtabs %}

### Operações set

As instruções intrínsecas de **set** definem o valor de todos os campos do registrador ao mesmo tempo sem a necessidade de usar uma _array_ para isso. Elas não são traduzidas para uma mas sim várias instruções em sequência, portanto pode haver uma penalidade de desempenho.

| Tecnologia | Protótipo                                                                                                                                                                     |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SSE2       | `__m128i _mm_set_epi16 (short e7, short e6, short e5, short e4, short e3, short e2, short e1, short e0)`                                                                      |
| SSE2       | `__m128i _mm_set_epi32 (int e3, int e2, int e1, int e0)`                                                                                                                      |
| SSE2       | `__m128i _mm_set_epi64 (__m64 e1, __m64 e0)`                                                                                                                                  |
| SSE2       | `__m128i _mm_set_epi64x (long long int e1, long long int e0)`                                                                                                                 |
| SSE2       | `__m128i _mm_set_epi8 (char e15, char e14, char e13, char e12, char e11, char e10, char e9, char e8, char e7, char e6, char e5, char e4, char e3, char e2, char e1, char e0)` |
| SSE2       | `__m128d _mm_set_pd (double e1, double e0)`                                                                                                                                   |
| SSE        | `__m128 _mm_set_ps (float e3, float e2, float e1, float e0)`                                                                                                                  |
| SSE2       | `__m128d _mm_set_sd (double a)`                                                                                                                                               |
| SSE        | `__m128 _mm_set_ss (float a)`                                                                                                                                                 |

As operações **set** escalares (`_mm_set_sd` e`_mm_set_ss`) definem o valor da parte menos significativa do registrador e zeram os demais valores.

As duas operações abaixo definem todos os campos do registrador para o mesmo valor passado como argumento:

| Tecnologia | Protótipo                        |
| ---------- | -------------------------------- |
| SSE2       | `__m128d _mm_set_pd1 (double a)` |
| SSE        | `__m128 _mm_set_ps1 (float a)`   |

Exemplo:

```c
#include <stdio.h>
#include <immintrin.h>

int main(void)
{
  __m128i data = _mm_set_epi32(444, 333, 222, 111);

  int number = _mm_extract_epi32(data, 2);
  printf("%d\n", number);
  return 0;
}
```

### Operações matemáticas

| Tecnologia | Protótipo                                      | Instrução                |
| ---------- | ---------------------------------------------- | ------------------------ |
| SSE2       | `__m128i _mm_add_epi16 (__m128i a, __m128i b)` | `paddw xmm, xmm`         |
| SSE2       | `__m128i _mm_add_epi32 (__m128i a, __m128i b)` | `paddd xmm, xmm`         |
| SSE2       | `__m128i _mm_add_epi64 (__m128i a, __m128i b)` | `paddq xmm, xmm`         |
| SSE2       | `__m128i _mm_add_epi8 (__m128i a, __m128i b)`  | `paddb xmm, xmm`         |
| SSE2       | `__m128d _mm_add_pd (__m128d a, __m128d b)`    | `addpd xmm, xmm`         |
| SSE        | `__m128 _mm_add_ps (__m128 a, __m128 b)`       | `addps xmm, xmm`         |
| SSE2       | `__m128d _mm_add_sd (__m128d a, __m128d b)`    | `addsd xmm, xmm`         |
| SSE2       | `__m64 _mm_add_si64 (__m64 a, __m64 b)`        | `paddq mm, mm`           |
| SSE        | `__m128 _mm_add_ss (__m128 a, __m128 b)`       | `addss xmm, xmm`         |
| SSE2       | `__m128d _mm_div_pd (__m128d a, __m128d b)`    | `divpd xmm, xmm`         |
| SSE        | `__m128 _mm_div_ps (__m128 a, __m128 b)`       | `divps xmm, xmm`         |
| SSE2       | `__m128d _mm_div_sd (__m128d a, __m128d b)`    | `divsd xmm, xmm`         |
| SSE        | `__m128 _mm_div_ss (__m128 a, __m128 b)`       | `divss xmm, xmm`         |
| SSE2       | `__m128d _mm_mul_pd (__m128d a, __m128d b)`    | `mulpd xmm, xmm`         |
| SSE        | `__m128 _mm_mul_ps (__m128 a, __m128 b)`       | `mulps xmm, xmm`         |
| SSE2       | `__m128d _mm_mul_sd (__m128d a, __m128d b)`    | `mulsd xmm, xmm`         |
| SSE        | `__m128 _mm_mul_ss (__m128 a, __m128 b)`       | `mulss xmm, xmm`         |
| SSE2       | `__m64 _mm_mul_su32 (__m64 a, __m64 b)`        | `pmuludq mm, mm`         |
| SSE2       | `__m128d _mm_sqrt_pd (__m128d a)`              | `sqrtpd xmm, xmm`        |
| SSE        | `__m128 _mm_sqrt_ps (__m128 a)`                | `sqrtps xmm, xmm`        |
| SSE2       | `__m128d _mm_sqrt_sd (__m128d a, __m128d b)`   | `sqrtsd xmm, xmm`        |
| SSE        | `__m128 _mm_sqrt_ss (__m128 a)`                | `sqrtss xmm, xmm`        |
| SSE2       | `__m128i _mm_sub_epi16 (__m128i a, __m128i b)` | `psubw xmm, xmm`         |
| SSE2       | `__m128i _mm_sub_epi32 (__m128i a, __m128i b)` | `psubd xmm, xmm`         |
| SSE2       | `__m128i _mm_sub_epi64 (__m128i a, __m128i b)` | `psubq xmm, xmm`         |
| SSE2       | `__m128i _mm_sub_epi8 (__m128i a, __m128i b)`  | `psubb xmm, xmm`         |
| SSE2       | `__m128d _mm_sub_pd (__m128d a, __m128d b)`    | `subpd xmm, xmm`         |
| SSE        | `__m128 _mm_sub_ps (__m128 a, __m128 b)`       | `subps xmm, xmm`         |
| SSE2       | `__m128d _mm_sub_sd (__m128d a, __m128d b)`    | `subsd xmm, xmm`         |
| SSE2       | `__m64 _mm_sub_si64 (__m64 a, __m64 b)`        | `psubq mm, mm`           |
| SSE        | `__m128 _mm_sub_ss (__m128 a, __m128 b)`       | `subss xmm, xmm`         |
| SSSE3      | `__m128i _mm_abs_epi16 (__m128i a)`            | `pabsw xmm, xmm`         |
| SSSE3      | `__m128i _mm_abs_epi32 (__m128i a)`            | `pabsd xmm, xmm`         |
| SSSE3      | `__m128i _mm_abs_epi8 (__m128i a)`             | `pabsb xmm, xmm`         |
| SSSE3      | `__m64 _mm_abs_pi16 (__m64 a)`                 | `pabsw mm, mm`           |
| SSSE3      | `__m64 _mm_abs_pi32 (__m64 a)`                 | `pabsd mm, mm`           |
| SSSE3      | `__m64 _mm_abs_pi8 (__m64 a)`                  | `pabsb mm, mm`           |
| SSE4.1     | `__m128d _mm_ceil_pd (__m128d a)`              | `roundpd xmm, xmm, imm8` |
| SSE4.1     | `__m128 _mm_ceil_ps (__m128 a)`                | `roundps xmm, xmm, imm8` |
| SSE4.1     | `__m128d _mm_ceil_sd (__m128d a, __m128d b)`   | `roundsd xmm, xmm, imm8` |
| SSE4.1     | `__m128 _mm_ceil_ss (__m128 a, __m128 b)`      | `roundss xmm, xmm, imm8` |
| SSE4.1     | `__m128d _mm_floor_pd (__m128d a)`             | `roundpd xmm, xmm, imm8` |
| SSE4.1     | `__m128 _mm_floor_ps (__m128 a)`               | `roundps xmm, xmm, imm8` |
| SSE4.1     | `__m128d _mm_floor_sd (__m128d a, __m128d b)`  | `roundsd xmm, xmm, imm8` |
| SSE4.1     | `__m128 _mm_floor_ss (__m128 a, __m128 b)`     | `roundss xmm, xmm, imm8` |
| SSE2       | `__m128i _mm_max_epi16 (__m128i a, __m128i b)` | `pmaxsw xmm, xmm`        |
| SSE4.1     | `__m128i _mm_max_epi32 (__m128i a, __m128i b)` | `pmaxsd xmm, xmm`        |
| SSE4.1     | `__m128i _mm_max_epi8 (__m128i a, __m128i b)`  | `pmaxsb xmm, xmm`        |
| SSE4.1     | `__m128i _mm_max_epu16 (__m128i a, __m128i b)` | `pmaxuw xmm, xmm`        |
| SSE4.1     | `__m128i _mm_max_epu32 (__m128i a, __m128i b)` | `pmaxud xmm, xmm`        |
| SSE2       | `__m128i _mm_max_epu8 (__m128i a, __m128i b)`  | `pmaxub xmm, xmm`        |
| SSE2       | `__m128d _mm_max_pd (__m128d a, __m128d b)`    | `maxpd xmm, xmm`         |
| SSE        | `__m64 _mm_max_pi16 (__m64 a, __m64 b)`        | `pmaxsw mm, mm`          |
| SSE        | `__m128 _mm_max_ps (__m128 a, __m128 b)`       | `maxps xmm, xmm`         |
| SSE        | `__m64 _mm_max_pu8 (__m64 a, __m64 b)`         | `pmaxub mm, mm`          |
| SSE2       | `__m128d _mm_max_sd (__m128d a, __m128d b)`    | `maxsd xmm, xmm`         |
| SSE        | `__m128 _mm_max_ss (__m128 a, __m128 b)`       | `maxss xmm, xmm`         |
| SSE2       | `__m128i _mm_min_epi16 (__m128i a, __m128i b)` | `pminsw xmm, xmm`        |
| SSE4.1     | `__m128i _mm_min_epi32 (__m128i a, __m128i b)` | `pminsd xmm, xmm`        |
| SSE4.1     | `__m128i _mm_min_epi8 (__m128i a, __m128i b)`  | `pminsb xmm, xmm`        |
| SSE4.1     | `__m128i _mm_min_epu16 (__m128i a, __m128i b)` | `pminuw xmm, xmm`        |
| SSE4.1     | `__m128i _mm_min_epu32 (__m128i a, __m128i b)` | `pminud xmm, xmm`        |
| SSE2       | `__m128i _mm_min_epu8 (__m128i a, __m128i b)`  | `pminub xmm, xmm`        |
| SSE2       | `__m128d _mm_min_pd (__m128d a, __m128d b)`    | `minpd xmm, xmm`         |
| SSE        | `__m64 _mm_min_pi16 (__m64 a, __m64 b)`        | `pminsw mm, mm`          |
| SSE        | `__m128 _mm_min_ps (__m128 a, __m128 b)`       | `minps xmm, xmm`         |
| SSE        | `__m64 _mm_min_pu8 (__m64 a, __m64 b)`         | `pminub mm, mm`          |
| SSE2       | `__m128d _mm_min_sd (__m128d a, __m128d b)`    | `minsd xmm, xmm`         |
| SSE        | `__m128 _mm_min_ss (__m128 a, __m128 b)`       | `minss xmm, xmm`         |
| SSE2       | `__m128i _mm_avg_epu16 (__m128i a, __m128i b)` | `pavgw xmm, xmm`         |
| SSE2       | `__m128i _mm_avg_epu8 (__m128i a, __m128i b)`  | `pavgb xmm, xmm`         |
| SSE        | `__m64 _mm_avg_pu16 (__m64 a, __m64 b)`        | `pavgw mm, mm`           |
| SSE        | `__m64 _mm_avg_pu8 (__m64 a, __m64 b)`         | `pavgb mm, mm`           |

Exemplos:

{% tabs %}
{% tab title="add_int.c" %}
```c
#include <stdio.h>
#include <immintrin.h>

int main(void)
{
  int array[4];
  __m128i data1 = _mm_set_epi32(444, 333, 222, 111);
  __m128i data2 = _mm_set_epi32(111, 222, 333, 444);

  __m128i result = _mm_add_epi32(data1, data2);
  _mm_store_si128((__m128i *)array, result);
  printf("%d, %d, %d, %d\n", array[0], array[1], array[2], array[3]);
  return 0;
}
```
{% endtab %}

{% tab title="sqrt_double.c" %}
```c
#include <stdio.h>
#include <immintrin.h>

int main(void)
{
  double array[2];
  __m128d data = _mm_set_pd(81.0, 625.0);

  __m128d result = _mm_sqrt_pd(data);
  _mm_store_pd(array, result);
  printf("%f, %f\n", array[0], array[1]);
  return 0;
}
```
{% endtab %}
{% endtabs %}

### Operações de randomização

As instruções intrínsecas abaixo leem um valor aleatório gerado por hardware:

| Tecnologia | Protótipo                                    | Instrução    |
| ---------- | -------------------------------------------- | ------------ |
| RDRAND     | `int _rdrand16_step (unsigned short* val)`   | `rdrand r16` |
| RDRAND     | `int _rdrand32_step (unsigned int* val)`     | `rdrand r32` |
| RDRAND     | `int _rdrand64_step (unsigned __int64* val)` | `rdrand r64` |

A instrução `rdrand` escreve o valor aleatório obtido no ponteiro passado como argumento. Ela deve ser usada em um _loop_ pois não há garantia de que ela irá obter de fato um valor. Se obter o valor a função retorna `1`, caso contrário retorna `0`.

Exemplo:

```c
#include <stdio.h>
#include <immintrin.h>

int main(void)
{
  int value;

  while (!_rdrand32_step(&value))
    ;

  printf("Valor: %d\n", value);
  return 0;
}
```

Você **deve** compilar passando a _flag_ `-mrdrnd` para o GCC para indicar que o processador suporta a tecnologia. Caso contrário você obterá um erro como este:

> error: inlining failed in call to always\_inline ‘\_rdrand32\_step’: **target specific option mismatch**

As instruções intrínsecas abaixo são utilizadas da mesma maneira que **rdrand** porém o valor aleatório não é gerado por hardware.

| Tecnologia | Protótipo                                     | Instrução    |
| ---------- | --------------------------------------------- | ------------ |
| RDSEED     | `int _rdseed16_step (unsigned short * val)`   | `rdseed r16` |
| RDSEED     | `int _rdseed32_step (unsigned int * val)`     | `rdseed r32` |
| RDSEED     | `int _rdseed64_step (unsigned __int64 * val)` | `rdseed r64` |

É necessário compilar com a _flag_ `-mrdseed` para poder usar essas instruções intrínsecas.
