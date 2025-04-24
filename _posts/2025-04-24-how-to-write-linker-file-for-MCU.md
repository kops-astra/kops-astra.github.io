---
title: "How to Write a Linker File for a Microcontroller Unit (MCU)"
date: 2025-04-24 00:00:00 +0530
categories: [design]
tags: [how to]
---

# How to Write a Linker File for a Microcontroller Unit (MCU)

In embedded systems development, writing a custom linker script is essential for defining how memory is used by your program. Unlike application software on desktops, embedded software runs in constrained environments where every byte of memory matters. A linker file (also known as a **linker script**) tells the linker where and how to place code and data in the MCU's memory map.

This article explains how to write a basic linker script for an MCU, typically used with toolchains like **GNU Arm Embedded Toolchain (GCC)**.

---

## 🔧 What Is a Linker Script?

A **linker script** is a configuration file used by the linker (`ld` in GCC) to describe:

- **Memory regions** of the MCU (e.g., Flash, RAM)
- **Placement of program sections** like `.text`, `.data`, `.bss`
- **Stack and heap size**
- **Entry point** (where execution starts)

This file usually has a `.ld` extension.

---

## 🧠 Understanding MCU Memory Layout

Before writing a linker script, you must know your microcontroller’s memory layout. For example, a typical ARM Cortex-M MCU might have:

- **Flash**: starting at `0x08000000`, size 256 KB
- **SRAM**: starting at `0x20000000`, size 64 KB

Check the MCU datasheet or reference manual for this information.

---

## 📝 A Basic Linker Script Example

Below is a basic linker script for an ARM Cortex-M MCU:

```ld
/* Simple linker script for STM32F103 (256KB Flash, 64KB RAM) */

ENTRY(Reset_Handler)  /* Entry point of the program */

MEMORY
{
  FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 256K
  RAM   (rwx): ORIGIN = 0x20000000, LENGTH = 64K
}

SECTIONS
{
  /* Code section */
  .text :
  {
    KEEP(*(.isr_vector))         /* Interrupt vector table */
    *(.text*)                    /* Application code */
    *(.rodata*)                  /* Read-only data */
    _etext = .;                  /* End of text */
  } > FLASH

  /* Initialized data section */
  .data : AT (ADDR(.text) + SIZEOF(.text))
  {
    _sdata = .;
    *(.data*)                    /* Initialized variables */
    _edata = .;
  } > RAM

  /* Uninitialized data section */
  .bss :
  {
    _sbss = .;
    *(.bss*)                     /* Uninitialized variables */
    *(COMMON)
    _ebss = .;
  } > RAM

  /* Stack section */
  ._stack (COPY):
  {
    . = ALIGN(8);
    _stack_start = .;
    . = . + 2K;                 /* 2KB stack size */
    _stack_end = .;
  } > RAM

  /* Heap section (optional) */
  .heap (COPY):
  {
    . = ALIGN(8);
    _heap_start = .;
    . = . + 4K;                 /* 4KB heap size */
    _heap_end = .;
  } > RAM
}
```

## 🧩 Key Components Explained
`MEMORY`
Defines the available memory regions. Each region has attributes:

`r`: readable
`w`: writable
`x`: executable

`SECTIONS`
Defines how different parts of your program map into memory regions:

`.text`: code and read-only constants, usually goes in Flash
`.data`: initialized data, copied from Flash to RAM during startup
`.bss`: uninitialized data, allocated in RAM
`stack` and `heap` are optional but critical for dynamic memory and function calls

`ENTRY(Reset_Handler)`
Tells the linker where program execution should start. This symbol must match the startup code.

---

## 🛠️ How to Use the Linker Script
When compiling with GCC, you pass the script to the linker:

```bash
arm-none-eabi-gcc -T my_linker_script.ld -nostartfiles -Wl,-Map=output map -o main.elf main.o startup.o
```
`-T`: Specifies the linker script
`-nostartfiles`: Prevents linking default startup code
`-Wl,-Map`: Generates a memory map file for inspection

---

## 🧪 Verifying the Memory Map
After building, inspect the .map file:

```bash
cat output.map | less
```
This shows where each section was placed and how much memory it uses. Make sure nothing overlaps or exceeds limits.

## 🧠 Tips for Writing Linker Scripts
- Align sections properly to avoid hard faults (e.g., 4- or 8-byte alignment).
- Use `KEEP()` for important sections like the interrupt vector table to prevent them from being discarded by the linker.
- Label start and end of sections (e.g., `_sdata`, `_edata`) for use in startup code.
- Avoid overlapping memory regions — always double-check addresses and sizes.

Keep the linker script version-controlled, just like code.

## 🧰 When Should You Modify the Linker Script?
You might need to modify or write your own linker script when:

- Using a custom board or memory layout
- Creating bootloaders or placing code at non-standard locations
- Reserving areas for non-volatile storage
- Partitioning memory for RTOS or secure zones

--- 

## 📦 Advanced: Placing Code or Data in Custom Sections
Want to place a function or variable in a specific memory region?

In code:
```c
__attribute__((section(".my_section")))
const char my_data[] = "Hello!";
```

In the linker script:
```ld
.my_section :
{
  *(.my_section)
} > FLASH
```
---

## Conclusion
Writing a linker script gives you precise control over how your firmware uses the microcontroller's memory. It’s a crucial skill for embedded developers, especially when working on bare-metal systems or performance-critical applications.

By mastering linker scripts, you not only avoid common memory pitfalls but also unlock advanced capabilities like multi-region bootloaders, RTOS memory isolation, and secure firmware updates.