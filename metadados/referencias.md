# Referências

## Bibliografia

### Arquiteturas x86 e x86-64

1. [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html)
2. [x86 and amd64 instruction reference - Félix Cloutier](https://www.felixcloutier.com/x86/)
3. [X86 Opcode and Instruction Reference](http://ref.x86asm.net/)
4. [AT&T Assembly Syntax](https://csiflabs.cs.ucdavis.edu/~ssdavis/50/att-syntax.htm)

### Convenções de chamada

1. [System V Application Binary Interface Intel386 Architecture Processor Supplement - Version 1.0](https://www.uclibc.org/docs/psABI-i386.pdf)
2. [System V Application Binary Interface AMD64 Architecture Processor Supplement - Draft Version 0.99.7](https://www.uclibc.org/docs/psABI-x86_64.pdf)
3. [Calling Conventions \| Microsoft Docs](https://docs.microsoft.com/en-us/cpp/cpp/calling-conventions)
4. [x86 calling conventions \| Wikipedia](https://en.wikipedia.org/wiki/X86_calling_conventions)

### Ferramentas

1. [Using as \| Documentation for binutils 2.37](https://sourceware.org/binutils/docs/as/)
2. [GAS syntax - Wikibooks](https://en.wikibooks.org/wiki/X86_Assembly/GAS_Syntax)
3. [NASM version 2.15.05 documentation](https://www.nasm.us/xdoc/2.15.05/html/nasmdoc0.html)
4. [Using the GNU Compiler Collection \(GCC\) \| GNU Project](https://gcc.gnu.org/onlinedocs/gcc/)

### Linguagem C

1. [C11 Standard - ISO/IEC 9899:201x draft n1570](http://www.open-std.org/jtc1/sc22/WG14/www/docs/n1570.pdf)
2. [The GNU C Reference Manual](https://www.gnu.org/software/gnu-c-manual/gnu-c-manual.html)
3. [GNU Compiler Collection \(GCC\) Internals](https://gcc.gnu.org/onlinedocs/gccint/)

### Linux

1. [Linux System Call Table for x86 64](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)
2. Linux Programmer's Manual
3. [Lazy binding](http://www.qnx.com/developers/docs/qnxcar2/topic/com.qnx.doc.neutrino.prog/topic/devel_Lazy_binding.html)
4. [The .init and .fini Sections](https://beefchunk.com/documentation/sys-programming/binary_formats/elf/elf_from_the_programmers_perspective/node3.html)

## Códigos consultados

Alguns trechos do livro foram baseados em conhecimento que obtive lendo diretamente o código-fonte de alguns projetos. Abaixo eu listo cada arquivo consultado para fins de referência. O `*` \(caractere curinga\) indica que consultei todos os arquivos de um determinado diretório.

### [glibc](https://sourceware.org/git/?p=glibc.git)

1. [/csu/\*](https://sourceware.org/git/?p=glibc.git;a=tree;f=csu;hb=refs/heads/master)
2. [/sysdeps/x86\_64/start.S](https://sourceware.org/git/?p=glibc.git;a=blob;f=sysdeps/x86_64/start.S;hb=refs/heads/master)
3. [/sysdeps/x86\_64/crti.S](https://sourceware.org/git/?p=glibc.git;a=blob;f=sysdeps/x86_64/crti.S;hb=refs/heads/master)

