---
tags:
  - Arch
  - TODO
  - OS
aliases:
  - CALL
  - ELF
---

# Compilation Steps

![compilation steps|500](compilation%20steps.png)

A `GCC`, provides a compiler driver that invokes the language *preprocessor*, *compiler*, *assembler*, and *linker*, as needed on behalf of the user:

1. The driver first runs the C *preprocessor* `cpp` which translates the C source file `hello.c` into an intermediate file `hello.i`
2. Next, the driver runs the C *compiler* `cc1`, which translates `hello.i` into an assembly file `hello.s`
3. Then, the driver runs the *assembler* `as`, which translates `hello.s` into a binary *relocatable object file* `hello.o`
4. Finally, it runs the *linker* `ld`, which combines `hello.o` and `printf.o`, along with the necessary system object files, to create the binary *executable object file* `hello`

# Linking

## Static Linking

![static linking|250](static%20linking.png)

Static linker `ld` takes as input relocatable object files and generates as output a fully linked executable object file that can be loaded and run

To build the executable, the linker must perform two main tasks:

1. **Symbol resolution** - associating each *symbol reference* with exactly one *symbol definition*
2. **Relocation** - associating a memory location with each symbol definition, and then modifying all of the references to those symbols so that they point to this memory location. Relocation is performed using *relocation entries*, generated by the assembler

### Linking with Static Libraries

![linking with static libraries|450](linking%20with%20static%20libraries.png)

Static library is a collection of relocatable object files, with a header that describes the size and location of each member object file. Static library is stored in an archive denoted with the `.a` suffix

The linker copies only the object files in the library that are referenced by the application program, which reduces the size of the executable on disk and in memory. The programmer only needs to include the names of a few library files instead of all required object files

Static libraries have some disadvantages:

- Programs must be explicitly relink against the updated libraries
- At run time, code for common functions (e.g. `printf`, `scanf`) is duplicated in the text segment of each running process

## Dynamic Linking with Shared Libraries

![dynamic linking|350](dynamic%20linking.png)

Shared library (shared object) is an object file that, at either run time or load time, can be loaded at an arbitrary memory address and linked with a program in memory. Shared library is denoted by the `.so` suffix

Shared libraries are "shared" in two different ways:

1. There is exactly one `.so` file for a particular library in a file system. The code and data in this `.so` file are shared by all of the executable object files that reference the library, as opposed to the contents of static libraries, which are copied and embedded in the executables that reference them
2. A single copy of the `.text` section of a shared library in memory can be shared by different running processes

Dynamic linking process:

1. Loader [loads](#Program%20Loading) the partially linked executable
2. Loader loads and runs the dynamic linker using the name from the `.interp` section
3. Dynamic linker performs relocations for the executable file and its shared objects
   Linker uses data in `.dynamic`, `.plt` and `.got` sections
4. Dynamic linker transfers control to the program

In Linux dynamic linker is a shared object `ld-linux.so` and is loaded as position-independent code; the system creates its segments in the dynamic segment area used by `mmap`

### Position-Independent Code (PIC)

#todo

A key purpose of shared libraries is to allow multiple running processes to share the same library code in memory. To allow that, the code segments of shared modules are compiled so that they can be loaded anywhere in memory without having to be modified by the linker

Code that can be loaded without needing any relocations is known as position-independent code (PIC)

To produce PIC compilers generate a global offset table (GOT) in `.got` section and a procedure linkage table (PLT) in `.plt` section, used by the dynamic linker and executable

# Object Files (ELF)

Created by the assembler and linker, object files are binary representations of programs intended to execute directly on a CPU

Linux uses Executable and Linkable Format (ELF) for object files, specified in [System V ABI](Application%20Binary%20Interface%20(ABI).md)

There are three types of object files:

1. **Relocatable object file** contains code and data suitable for linking with other object files to create an executable or a shared object file
2. **Shared object file** contains code and data suitable for linking in two contexts
	1. Linker may process it with other relocatable and shared object files to create another object file
	2. Dynamic linker combines it with an executable file and other shared objects to create a process image
3. **Executable object file** contains code and data in a form that can be copied directly into memory and executed

## ELF Header

Describes the file organization

Important fields are:

- `e_type`: Object file type
	- `ET_REL`: Relocatable object file
	- `ET_EXEC`: Executable object file
	- `ET_DYN`: Shared object file
- `e_machine`: Target architecture (e.g. x86-64)
- `e_entry`: Entry point's [virtual address](Virtual%20Memory.md)
- `e_phoff`: Program header table file offset
- `e_shoff`: Section header table file offset
- `e_ehsize`: Size (in bytes) of the ELF header
- `e_shentsize`: Size of a section header table entry
- `e_shnum`: Number of entries in the section header table
- `e_phentsize`: Size of a program header table entry
- `e_phnum`: Number of entries in the program header table

## Program Header Table

Provides a *segment view*, describing a mapping of contiguous chunks of the executable file to contiguous memory segments. It is used by the OS and dynamic linker when loading an ELF into a memory for execution

Executable and shared object files must have a program header table; relocatable files do not need one

An object file segment contains one or more sections. For example, the code segment contains `.init`, `.text`, `.rodata` sections; the data segment contains `.data`, `.bss` sections

Each header contains the following:

- `p_type`: Type of the segment
	- `PT_LOAD`: Loadable segment, the bytes from the file are mapped to the beginning of the memory segment
	- `PT_DYNAMIC`: Dynamic linking information (holds the `.dynamic` section)
	- `PT_INTERP`: Runtime interpreter information (holds `.interp` section)
- `p_flags`: Segment flags
	- `PF_X`: Executable segment
	- `PF_W`: Writable segment
	- `PF_R`: Readable segment
- `p_offset`: Offset of the segment in the file image
- `p_vaddr`: Virtual address of the segment in memory
- `p_filesz`: Size (in bytes) of the segment in the file image
- `p_memsz`: Size (in bytes) of the segment in memory
- `p_align`: Segment alignment

## Section Header Table

Provides a *section view*, describing all the sections in the object file

Files used during linking must have a section header table; other object files may or may not have one

Each header contains the following:

- `sh_name`: An offset to a string in the `.shstrtab` section that represents the name of this section
- `sh_type`: Type of the section
	- `SHT_PROGBITS`: Program data (such as machine instructions or constants)
	- `SHT_SYMTAB`: Symbol table
	- `SHT_DYNSYM`: Dynamic linker symbol table
	- `SHT_STRTAB`: String table
	- `SHT_REL`: Relocation entries
	- `SHT_DYNAMIC`: Dynamic linking information
	- `SHT_NOBITS`: Uninitialized data
- `sh_flags`: Section flags
	- `SHF_WRITE`: Contains data that should be writable during execution
	- `SHF_ALLOC`: Occupies memory during process execution, not set for `.bss` section
	- `SHF_EXECINSTR`: Contains executable machine instructions
- `sh_addr`: Virtual address of the section in memory, for sections that are loaded
- `sh_offset`: Offset of the section in the file image
- `sg_size`: Size (in bytes) of the section in the file image
- `sh_link`: Link to another section
- `sh_addralign`: Section alignment
- `sh_entsize`: Size (in bytes) of each entry, for sections that contain entries (e.g. symbol table, relocation table)

## Sections

Some common sections:

- `.init`: Executable code that performs initialization tasks
- `.text`: The machine code of the compiled program
- `.rodata`: Read-only data such as the format strings in `printf` statements
- `.data`: Initialized writable data
- `.bss`: Uninitialized data. This section occupies no actual space in the object file
- `.rel.*`: Relocation information, e.g. a relocation section for `.text` is in `.rel.text`
- `.dynamic`: Dynamic linking structures and objects
- `.debug`: A debugging symbol table
- `.symtab`: A symbol table which associates a symbolic name to functions and global variables
- `.shstrtab`: A string table that contains the names of all the sections in the binary
- `.strtab`: A string table, containing symbols names, used by symbol tables in `.symtab` and `.debug` sections
- `.dynsym`: Same as `.symtab` but contains symbols needed for dynamic-linking
- `.dynstr`: Same as `.strtab` but contains strings needed for dynamic-linking
- `.interp`: Path name of the dynamic loader (interpreter)
- `.plt`: Procedure linkage table
- `.got`: Global offset table

## Auxiliary Vector

Auxiliary vector contains information from the OS about the environment in which it is operating. For example, entry with `AT_SYSINFO` type contains the pointer to the global system page used for [system calls](System%20Calls.md)

The primary customer of the auxiliary vector is the dynamic linker `ls-linux.so`

Auxiliary vector is an array of entries:

- `a_type`: Entry type
	- `AT_SYSINFO_EHDR`: Location of the [vDSO](System%20Calls.md) page on x86-64
	- `AT_EXECFN`: Location of the program filename
	- `AT_ENTRY`: Program entry point
- `a_un`: Value interpreted according to `a_type`

Loader puts auxiliary vector `auxv` on the process [stack](Call%20Stack.md) along with other information like `argc`, `argv` and `envp`

# Symbol Resolution

The purpose of symbol resolution is to associate each symbol reference with exactly one symbol definition

The linker resolves symbol references by associating each reference with exactly one symbol definition from the symbol tables of its input relocatable object files

## Symbol Table

Symbol tables are built by assemblers, using symbols exported by the compiler

Symbol table entry contains:

- `st_name`: Symbol name, byte offset into the string table in `.strtab` section
- `st_value`: Symbol address
	- For `ET_REL`, an offset from the beginning of the section that `st_shndx` identifies
	- For `ET_EXEC` and `ET_DYN`, a [virtual address](Virtual%20Memory.md)
- `st_size`: Size (in bytes) of the object
- `st_info`: Symbol type and binding (e.g. data or function, local or global)
- `st_shndx`: Index into the section header table, denoting the section this symbol is assigned to

# Relocation

Whenever the assembler encounters a reference to an object whose target location is unknown, it generates a relocation entry that tells the linker how to modify the reference when it merges the object files together

## Relocation Table

Relocation entries for code are placed in `.rel.text`. Relocation entries for data are placed in `.rel.data`

Relocation entry contains:

- `r_offset`: Relocation address
	- For `ET_REL`, an offset within a section in which the relocation have to take place
	- For `ET_EXEC` and `ET_DYN`, a [virtual address](Virtual%20Memory.md) affected by a relocation
- `r_info`: Both the symbol table index and the relocation type. Some common types are:
	- `R_X86_64_PC*`: Relocate a reference that uses a `PC`-relative address
	- `R_X86_64_*`: Relocate a reference that uses an absolute address

# Program Loading

![Linux x86-64 program memory layout|300](Linux%20x86-64%20program%20memory%20layout.png)

Linux program runs in the context of a process with its own [virtual address space](Virtual%20Memory.md)

Loading and running the dynamically linked program comprises of the following steps:

1. When the shell runs a program, the parent shell process forks a child process that is a duplicate of the parent
2. The child process invokes the loader via the `execve()` [system call](System%20Calls.md)
3. The loader deletes the child's existing virtual memory segments
4. Loader sets up the virtual memory for the new program with ASLR support
	1. Maps the program file `PT_LOAD` segments into the process's address space, setting up the new program's memory layout
	2. Maps the [vDSO](System%20Calls.md) into the virtual address space
	3. Populates the [auxiliary vector](#Auxiliary%20Vector)
		1. The `AT_EXECFN` value holds the location of the program filename
		2. The `AT_ENTRY` value holds the address of the program entry point (`_start()` function), initialized by the kernel from the `e_entry` ELF header field
	4. Sets up the [stack](Call%20Stack.md) by inserting the data to the new program's stack
		1. The argument count `argc` and arguments `argv`
		2. Environment variables `expv`
		3. The populated auxiliary vector `auxv`
5. Loads interpreter, specified by the `PT_INTERP` program header, into memory similar to the program loading described above
6. Sets the entry point to the interpreter entry point, rather than that of the program itself
7. Loader returns from the `execve()` system call
8. Interpreter (dynamic linker) performs dynamic linking — finding and loading the shared libraries, and resolving the program's undefined symbols to the correct definitions in those libraries
9. Interpreter jumps to the `_start()` function (at the address recorded in `AT_ENTRY` auxiliary value), defined in the system object file `crt1.o`
10. The `_start()` function calls the system startup function, `__libc_start_main()`, which is defined in `libc.so`
11. It initializes the execution environment and calls the application `main()` function

Aside from some header information, there is no copying of data from disk to memory during loading. A process does not require a [physical page](Virtual%20Memory.md) unless it references the logical page during execution, at which point the OS transfers the page from disk to memory using its paging mechanism. Thus, executable and shared object files must have segment images whose file offsets and virtual addresses are congruent, modulo the page size

## Address Space Layout Randomization (ASLR)

With ASLR, different parts of the program, including program code, library code, [stack](Call%20Stack.md), global variables, and [heap data](Heap%20Memory.md), are loaded into different regions of memory each time a program is run. That means that a program running on one machine will have very different address mappings than the same program running on other machines, which eliminate some form of attacks

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [Dive Into Systems A Gentle Introduction to Computer Systems. Suzanne J. Matthews, Tia Newhall, Kevin C. Webb](References.md#Dive%20Into%20Systems%20A%20Gentle%20Introduction%20to%20Computer%20Systems.%20Suzanne%20J.%20Matthews,%20Tia%20Newhall,%20Kevin%20C.%20Webb)
- [Specification ELFs.pdf](https://refspecs.linuxfoundation.org/elf/elf.pdf)
- [ELF Format Cheatsheet · GitHub](https://gist.github.com/x0nu11byt3/bcb35c3de461e5fb66173071a2379779)
- [Executable and Linkable Format - Wikipedia](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)
- [CMU 15-213: Linking - YouTube](https://youtu.be/ZbKImUe3mQs?si=2MioSHinsEaPpykd)
- [HW3 - 238P Operating Systems: The ELF Format](https://ics.uci.edu/~aburtsev/238P/hw/hw3-elf/hw3-elf.html)
- [Position-independent code - Wikipedia](https://en.wikipedia.org/wiki/Position-independent_code)
- [getauxval() and the auxiliary vector \[LWN.net\]](https://lwn.net/Articles/519085/)
- [Auxiliary Vector (The GNU C Library)](https://www.gnu.org/software/libc/manual/html_node/Auxiliary-Vector.html)
- [How programs get run \[LWN.net\]](https://lwn.net/Articles/630727/)
- [How programs get run: ELF binaries \[LWN.net\]](https://lwn.net/Articles/631631/)