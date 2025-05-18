<!---
{
  "depends_on": [],
  "author": "Stephan Bökelmann",
  "first_used": "2025-04-01",
  "keywords": ["assembly", "GAS", "syscall", "x86_64", "Linux"]
}
--->

# Introduction to GAS on Linux

## 1) Introduction

Assembly programming allows us to interact with the **bare metal** of the machine. Using **GAS (GNU Assembler)**, a popular assembler for x86 and x86_64 architectures, we can write low-level code that speaks almost directly to the CPU.

GAS is:  
- The GNU Assembler, typically invoked as `as`, and part of the GNU Binutils suite.  
- Used to assemble assembly language source files into machine code for various architectures.  
- Compatible with the AT&T syntax by default, though it can also support Intel syntax via directives.  
- Often used as the backend assembler by the GNU Compiler Collection (GCC) to produce object files.  
- Cross-platform and supports cross-compilation for different target architectures.  
- Uses a two-pass assembly process: the first to parse and resolve symbols, the second to generate machine code.  
- Supports macros and conditional assembly using `.macro`, `.if`, `.endif`, etc.  
- Produces relocatable object files (typically `.o`) that can be linked using the GNU linker `ld`.  
- Supports various debugging formats including DWARF and STABS for integration with tools like GDB.  


In this exercise, we will write a minimal GAS program that calls the Linux `exit` syscall to terminate a process with a specific return code. This teaches how system calls work, how arguments are passed via registers, and how the program execution begins at `_start`.

To install GAS on your Linux system:

```bash
sudo apt update
sudo apt install binutils
```

Verify installation:

```bash
as -v
```

Here is your first complete GAS program:

```as
.section .text
.globl _start

_start:
    mov    $60, %rax       # syscall number for exit (60)
    mov    $42, %rdi       # exit code 42
    syscall                # invoke kernel
```

To assemble and run it:

```bash
$ as -o exit.o exit.s
$ ld -o exit exit.o
$ ./exit
$ echo $?
```

This is a minimal, but powerful example that demonstrates the Linux syscall ABI: parameters go into specific registers (`rdi`, `rsi`, `rdx`…), the syscall number goes into `rax`, and the `syscall` instruction triggers the transition into kernel space.

Whenever the `syscall` instruction is encounterd in an assembly-program, the processor changes its behavior. Instead of further running in `ring 3` or _User Mode_ the processor clears its _Instruction Cache_, switches into `ring 0` or _Kernel Mode_ and loads the next instructions from a location called **Model-Specific Register / (Long System Target Address Register** or short: `MSR_LSTAR`.

This is a special address, which is set during the boot process. 
The Kernel than takes care of the instructions that were written into the registers (`rdi`, `rsi`, `rdx`…).

## 2) Tasks

1. **Install GAS**: Install `gas` via your package manager by installing `binutils` and verify it works.
2. **Exit with Code 7**: Modify the program above to exit with code 7 instead of 42.
3. **Invalid Syscall**: Replace `mov rax, 60` with an invalid syscall number. Observe the behavior!

## 3) Questions

1. What does the `globl _start` directive do? .globl_start defines _start
 as a global symbol. It#s needed for the linker, so that he knows where the function starts.
2. Why do we use `rax` to specify the syscall? %rax is used for the Kernal functions, for example %60 is used to delete the last process, wich causes to kill the actual process.
3. What would happen if you omit the `syscall` instruction? The core would dump again.
4. What is the difference between the `ld` and `as` steps? The "as" Step does take the asambley code and put it to machine code, wich is stored in an .o data. The .o file is not executable. The linker stepp does link a few .o files to an executable programm.

<details>
  <summary>Hint: syscall numbers</summary>

  Check out the Linux syscall table for x86_64:  
  https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/
</details>

## 4) Advice

Learning assembly is not about writing large programs — it's about understanding the **machine** beneath your abstractions. Starting with a minimal syscall lets you bypass the C runtime, linker conveniences, and libc entirely. This helps to demystify what's really going on when you run a program.

Don’t be afraid to break things. Inspect binaries, modify instructions, and read system call tables. In the words of Donald Knuth:

> “People who are more than casually interested in computers should have at least some idea of what the underlying hardware is like.”  
