---
description: Entendendo a execução de código em C no ambiente freestanding.
---

# Ambiente freestanding

O ambiente de execução _freestanding_ é normalmente usado quando o código C é compilado para executar fora de um sistema operacional. Nesse ambiente nenhum dos recursos provindos do ambiente _hosted_ são garantidos e sua existência ou não depende da implementação.

Os únicos recursos que são oferecidos pela libc são os declarados nos seguintes _header files_:

**\<float.h>**, **\<iso646.h>**, **\<limits.h>**, **\<stdalign.h>**, **\<stdarg.h>**, **\<stdbool.h>**, **\<stddef.h>**, **\<stdint.h>** e **\<stdnoreturn.h>**.

Quaisquer outros recursos são dependentes de implementação.

No GCC para compilar um código visando o ambiente _freestanding_ é possível usar a opção `-ffreestanding`. Também se pode usar a opção `-fhosted` para compilar para ambiente _hosted_ mas esse já é o padrão.

Já a opção `-nostdlib` desabilita a _linkedição_ da libc.
