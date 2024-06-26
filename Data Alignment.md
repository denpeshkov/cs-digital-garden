---
tags:
  - Arch
---

# Alignment Requirement

A memory address $A$ is **$N$-byte aligned** when $A$ is a multiple of $N$ (where $N$ is a power of 2)  
A data is **naturally aligned** when data's memory address is a multiple of the data size

The effects of performing an unaligned memory access vary from architecture to architecture:

- On some architectures unaligned accesses may not be supported
- Unaligned accesses may be [non-atomic](Atomic%20Instructions.md)
- Accesses that cross a [cache line](Cache%20Memory.md) boundary may cause a performance penalty because multiple lines need to be fetched
- Accesses that cross a [page](Virtual%20Memory.md) boundary may cause an even bigger performance penalty because multiple pages need to be fetched.
- A DRAM typically supports only aligned [bursts](Main%20Memory.md)

# Satisfying Alignment Requirements

Alignment is enforced by making sure that every data type is organized and allocated in such a way that every object within the type satisfies its alignment restrictions

- Initial address of the object must be aligned. Starting address for any block generated by a memory allocator must be aligned
- Every field inside the object must be aligned
- Size of the object must be aligned to satisfy alignment for arrays of objects
- Stack frame must be aligned

For example, given this struct:

```c
struct S {
	T1 f1
	T2 f2
	...
	Tn fn
}
```

Where `Tmax` is the size of the biggest type `T1`…`Tn`Compiler must ensure (by adding **paddings**) that:

- Any pointer `p` of type struct `S*` satisfies the alignment: `*p` is a multiple of `Tmax`
- Each element `fi` satisfies the alignment: `&fi` is a multiple of `Ti`
- Size of the `S` is a multiple of size of the biggest element's type

The compiler places directives `.align` in the assembly code indicating the desired alignment for global data

# References

- [Data structure alignment - Wikipedia](https://en.wikipedia.org/wiki/Data_structure_alignment)
- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [CS24-Sp18-Introduction-To-Computing-System - YouTube](https://youtube.com/playlist?list=PL3swII2vlVoXiqUBV524pKEsP1iBN4UBU&si=B_w5UOwuIXVU-pq_)
- [ICS-CMU - YouTube](https://youtube.com/playlist?list=PLbY-cFJNzq7z_tQGq-rxtq_n2QQDf5vnM&si=QpoXB3iC86pd7U9-)
- [assembly - Why is memory alignment needed? - Stack Overflow](https://stackoverflow.com/questions/46991418/why-is-memory-alignment-needed)
- [Intermation: Data Structure Alignment and Endianness - YouTube](https://youtu.be/zyGMyV955Rw?si=xRlrYhZU9FqPLYEv)
- [Dive Into Systems A Gentle Introduction to Computer Systems. Suzanne J. Matthews, Tia Newhall, Kevin C. Webb](References.md#Dive%20Into%20Systems%20A%20Gentle%20Introduction%20to%20Computer%20Systems.%20Suzanne%20J.%20Matthews,%20Tia%20Newhall,%20Kevin%20C.%20Webb)
- [c - CPU and Data alignment - Stack Overflow](https://stackoverflow.com/questions/3025125/cpu-and-data-alignment)
- [x86 64 - Why do align access and non-align access have same performance? - Stack Overflow](https://stackoverflow.com/questions/70424501/why-do-align-access-and-non-align-access-have-same-performance)
