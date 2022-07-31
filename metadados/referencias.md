# Referências

## Bibliografia

### Arquiteturas x86 e x86-64

1. [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html)
2. [x86 and amd64 instruction reference - Félix Cloutier](https://www.felixcloutier.com/x86/)
3. [X86 Opcode and Instruction Reference](http://ref.x86asm.net/)
4. [AT\&T Assembly Syntax](https://csiflabs.cs.ucdavis.edu/\~ssdavis/50/att-syntax.htm)

### Convenções de chamada

1. [System V Application Binary Interface Intel386 Architecture Processor Supplement - Version 1.0](https://www.uclibc.org/docs/psABI-i386.pdf)
2. [System V Application Binary Interface AMD64 Architecture Processor Supplement - Draft Version 0.99.7](https://www.uclibc.org/docs/psABI-x86\_64.pdf)
3. [Calling Conventions | Microsoft Docs](https://docs.microsoft.com/en-us/cpp/cpp/calling-conventions)
4. [x86 calling conventions | Wikipedia](https://en.wikipedia.org/wiki/X86\_calling\_conventions)

### Depuradores

1. [GDB Internals - Breakpoint Handling](https://sourceware.org/gdb/wiki/Internals/Breakpoint%20Handling)

### Ferramentas

1. [Using as | Documentation for binutils 2.37](https://sourceware.org/binutils/docs/as/)
2. [GAS syntax - Wikibooks](https://en.wikibooks.org/wiki/X86\_Assembly/GAS\_Syntax)
3. [NASM version 2.15.05 documentation](https://www.nasm.us/xdoc/2.15.05/html/nasmdoc0.html)
4. [Using the GNU Compiler Collection (GCC) | GNU Project](https://gcc.gnu.org/onlinedocs/gcc/)
5. [GNU Compiler Collection (GCC) Internals](https://gcc.gnu.org/onlinedocs/gccint/)
6. [Debugging with GDB](https://sourceware.org/gdb/current/onlinedocs/gdb/)

### Instruções intrínsecas

1. [Intrinsics | Intel® C++ Compiler Classic Developer Guide and Reference](https://software.intel.com/content/www/us/en/develop/documentation/cpp-compiler-developer-guide-and-reference/top/compiler-reference/intrinsics.html)
2. Intel® 64 and IA-32 Architectures Software Developer Manuals - Volume 2, Appendix C
3. [Intel® Intrinsics Guide](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)
4. [An Introduction to GCC Compiler Intrinsics in Vector Processing](https://www.linuxjournal.com/content/introduction-gcc-compiler-intrinsics-vector-processing)

### Linguagem C

1. [C11 Standard - ISO/IEC 9899:201x draft n1570](http://www.open-std.org/jtc1/sc22/WG14/www/docs/n1570.pdf)
2. [The GNU C Reference Manual](https://www.gnu.org/software/gnu-c-manual/gnu-c-manual.html)
3. Frederico Lamberti Pissarra. [**Dicas - C e Assembly para arquitetura x86-64**](https://www.mentebinaria.com.br/files/file/31-dicas-c-e-assembly-para-arquitetura-x86-64/)****

### Linux

1. [Linux System Call Table for x86 64](https://blog.rchapman.org/posts/Linux\_System\_Call\_Table\_for\_x86\_64/)
2. Linux Programmer's Manual
3. [Lazy binding](http://www.qnx.com/developers/docs/qnxcar2/topic/com.qnx.doc.neutrino.prog/topic/devel\_Lazy\_binding.html)
4. [The .init and .fini Sections](https://beefchunk.com/documentation/sys-programming/binary\_formats/elf/elf\_from\_the\_programmers\_perspective/node3.html)
5. [ptrace(2) — Linux manual page](https://man7.org/linux/man-pages/man2/ptrace.2.html)
6. [ld.so(8) — Linux manual page](https://man7.org/linux/man-pages/man8/ld.so.8.html)
7. Daniel P. Bovet, Marco Cesati. [**Understanding the Linux Kernel, 3rd Edition - 4.5 Exception Handling**](https://www.oreilly.com/library/view/understanding-the-linux/0596005652/ch04s05.html)****

### Sistemas Operacionais

1. Andrew S. Tanenbaum. **Sistemas Operacionais Modernos**. 4° Edição. ISBN: 978-8543005676
2. [Escalonamento de processos | Wikipédia](https://pt.wikipedia.org/wiki/Escalonamento\_de\_processos)
3. [Troca de contexto | Wikipédia](https://pt.wikipedia.org/wiki/Troca\_de\_contexto)
4. [Sinal (ciência da computação) | Wikipédia](https://pt.wikipedia.org/wiki/Sinal\_\(ci%C3%AAncia\_da\_computa%C3%A7%C3%A3o\))

## Códigos consultados

Alguns trechos do livro foram baseados em conhecimento que obtive lendo diretamente o código-fonte de alguns projetos. Abaixo eu listo cada arquivo consultado para fins de referência. O `*` (caractere curinga) indica que consultei todos os arquivos de um determinado diretório.

### [glibc](https://sourceware.org/git/?p=glibc.git)

1. [/csu/\*](https://sourceware.org/git/?p=glibc.git;a=tree;f=csu;hb=refs/heads/master)
2. [/sysdeps/x86\_64/start.S](https://sourceware.org/git/?p=glibc.git;a=blob;f=sysdeps/x86\_64/start.S;hb=refs/heads/master)
3. [/sysdeps/x86\_64/crti.S](https://sourceware.org/git/?p=glibc.git;a=blob;f=sysdeps/x86\_64/crti.S;hb=refs/heads/master)
4. [/sysdeps/x86\_64/crtn.S](https://sourceware.org/git/?p=glibc.git;a=blob;f=sysdeps/x86\_64/crtn.S;hb=refs/heads/master)
