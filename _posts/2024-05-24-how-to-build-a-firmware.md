---
title: "How to build a Firmware"
date: 2024-05-24 00:00:00 +0530
categories: [build]
tags: [how to]
---

The build process in embedded C development involves translating human-readable source code into machine-executable code that can run on a microcontroller or embedded system. This process typically includes several steps, and toolchains play a vital role in each stage. Hence, it is very vital to understand a bit more about toolchains before we get into the details of the build process.
## Toolchain
A toolchain is the set of tools that compiles source code into executables that can run on your target device, and includes a compiler, a linker, and run-time libraries. In simple terms, Toolchain is software that creates a binary executable file for the code you have written. It is this code that you flash in the microcontroller. <br>
However, an important distinction is that toolchain is not IDE. For example, Keil/VS-Code/Eclipse is an IDE whereas arm-elf-gcc/gcc-arm-none-eabi is a toolchain. Normally, IDEs have these toolchain inbuilt. If you are using Linux, you would have you run your toolchain independently. <br>
The choice of toolchain can significantly impact the efficiency and effectiveness of the firmware build process. Different toolchains may offer varying levels of optimization, support for different microcontroller architectures, integration with IDEs, debugging capabilities, and compatibility with third-party libraries and frameworks. Therefore, selecting the right toolchain is an important decision for embedded developers. <br>

Now, that we have an understanding of the toolchain lets get into the overview of the typical build process:<br>
# 1.	Writing Source Code:
Developers write C code using a text editor or an integrated development environment (IDE). This code contains the logic and functionality required for the embedded system. This code may include various modules, functions, data structures, and other constructs necessary to achieve the system's objectives. This source code can be written in languages like C, C++, or assembly language.
# 2. Compilation
## 1. Preprocessing:
The preprocessor is a tool that runs before the actual compilation process.  The preprocessor removes all the comments from the c file. The preprocessor expands macros, includes header files, and handles conditional compilation. This involves handling directives like #include, #define, and #ifdef. For example:
1. __#include:__ Inserts the contents of a header file into the source code.
2. __#define:__ Defines macros, which are often used for constants or code snippets that need to be repeated.
3. __#ifdef:__ Conditional compilation based on whether a certain macro is defined.



## 2. Compiler: 
The pre-processed source code is fed into a compiler, which translates it into mcu architecture specific assembly code. The compilation process has two parts: Front end Analysis and Back End Sytesis.
### 1.	Front End Analysis:
Front End Analysis checks for syntax and semantics of the code. It also resolves variable names, and maintains the information of the declared variables in a structure called symbol table which has its type, scope and so on.
### 2.	Back End Synthesis:
When there are no errors in the front end analysis the complier takes its next action, Back End Synthesis which involves optimization and code generation. There are many forms of optimization such as transforming code into smaller or faster but functionally equivalent, inline expansion of functions, dead code removal, loop unrolling, and register allocation. Then in code generation, the compiler converts the optimized intermediate code structure into assembly code(.s or .asm file) based on the MCU architecture.
<br>
## 3.	Assembly: 
The assembler translates these assembly codes generated in the pervious into machine code. Optionally, if you're working at a low level, you might have inline assembly code or separate assembly files. These are also compiled to get object files(*.o or *.obj file) using an assembler. An object file that contains opcodes and data sections. <br>
To be able to understand how the compiler allocates memory to the code and data, first we have to know the different memory segments and what they contain. <br>
The different memory sections are: <br>
__.text Segment:__ A text segment, also known as a code segment or simply as text, is one of the sections of a program in an object file or in memory, which contains executable instructions. <br>
__.data Segment:__ Initialized data segment, usually called simply the Data Segment. A data segment is a portion of virtual address space of a program, which contains the global variables and static variables that are initialized by the programmer. <br>
__.bss Segment:__ Uninitialized data starts at the end of the data segment and contains all global variables and static variables that are initialized to zero or do not have explicit initialization in source code. <br>
__stack Segment:__ The stack data segment stores temporary data like local variables and return addresses. <br>
__heap Segment:__ Dynamic data storage. <br>
The final output of assembler is an object file, to better understand what's going to happen in the next stage, we have to know what exactly inside that object file. <br>

__Code and data__ is .text,.data,.rodata,.bss, stack, heap section that forms the part of the code which you have written.
__Symbol table__ is used to store all variable names and their attributes.
__Debug info__ section that has the mapping between the original source code and the information needed by the debugger.
__Exports section__ contains global symbols either functions or variables.
__Imports section__ contains symbol names that are needed from other object files.
Exports, imports, and symbol table sections are used by the linker during linking stage.

## 4.	Linking: 
The linker takes all the object files generated by the compiler, along with any library files, and combines them into a single executable file. The linker resolves external references, assigns addresses to functions and variables, and generates a binary file (typically with extensions like .elf, .bin, or .hex) that can be flashed onto the microcontroller's memory.

While combining the object files together, the linker performs the following operations:

### 1.	Symbol resolution.
In multi-file program, if there are any references to labels defined in another file, the assembler marks these references as “unresolved”. When these files are passed to the linker, the linker determines the values of these references from other object files, and patches the code with the correct values. 

- If the linker didn’t find any references to these labels in any object file, it will throw and a linking error “unresolved reverence to variable”.

- If the linker finds same symbol defined in two object files, it will report a “redefinition” error.

### 2.	Relocation
Relocation is the process of changing addresses already assigned to labels. This will also involve patching up all references to reflect the newly assigned address. In each file, the linker is responsible for merging sections from the input files, into sections of the output file. By default, the sections, with the same name, from each file is placed contiguously and the label to references are patched to reflect the new address. 

### 3.	Locator
In this stage the process of assigning physical addresses to the relocatable file is performed using a locator. The input file for this stage relocatable file, and Linker script file. The output file is executable file. The linker script file provides the locator with the required information about the actual memory layout and then the locator performs the conversion to produce a single executable binary file. In the relocatable file each section is assumed to start from address 0. And thus, labels are assigned values relative to the start of the section. When the final executable is created, the section is placed at some address X defined in the linker file . And all references to the labels defined within the section, are incremented by X, so that they point to the new location. Note that the locator can be found as a separate tool or with the linker.

The final output of this stage will be an executable binary image called *.elf file , where elf stands for executable and link format. Basically, the GCC based compilers will generate an executable file of .elf. It’s a debug + executable file of our source code that is ready to be loaded to our embedded target.

## 5.	Generating Output Files: 
Depending on the project requirements, various output files may be generated during the build process. These could include executable binaries, srec, debug files, listing files, and map files.

## 6.	Flashing/Programming: 
Once the binary file is generated, it needs to be flashed or programmed onto the target microcontroller's non-volatile memory. This process can be done using hardware tools like JTAG, SWD, or via bootloader mechanisms, depending on the specific microcontroller and development setup.

## 7.	Debugging and Testing: 
After flashing the program onto the microcontroller, developers perform testing and debugging to ensure that the code behaves as expected. This may involve using debugging tools, such as hardware debuggers or software emulators, to step through the code and identify and fix any issues.

Throughout this process, it's crucial to consider factors like optimization, memory usage, and real-time constraints, as embedded systems often have limited resources compared to traditional computing environments. Additionally, the build process may be automated using build scripts or Makefiles to streamline development and ensure consistency across different builds.
