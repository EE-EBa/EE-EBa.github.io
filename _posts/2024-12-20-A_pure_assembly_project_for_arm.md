---
title: A pure assmebly project for arm
date: 2024-12-20
categories: [Embedded, Assmebly]
tags: [assmebly]     # TAG names should always be lowercase



# 以下默认false
math: true
mermaid: true
pin: true

# 以下默认ture
toc: false
comments: false
---

# Creat project directionary

```bash
mkdir bareStm32
cd bareStm32
code main.s
```

# Write main.s file
This file need to point to assmebly's syntax , mcu type , instruction sets type etc.<br>
```asm
.syntax unified 
.cpu cortex-m0plus
.fpu softvfp
.thumb

.global g_pfnVectors
.global Reset_Handler
Reset_Handler:
  ldr   r0 , =0x50000800+0x0c
  ldr   r1 , [r0]
  movs  r2 , #(1 << 0)
  lsls  r2 , #16
  orrs  r1 , r1 , r2
  str   r1 , [r0]


  ldr   r0 , =0x50000800+0x0c
  ldr   r1 , [r0]
  movs  r2 , #0
  lsls  r2 , #16
  bics  r1 , r1 , r2
  str   r1 , [r0]

```
# Creat .ld (link script file) 

```ld
/*Entry Point*/
ENTRY(Reset_Handler)

MEMORY
{
    flash(rx)   : ORIGIN =0x08000000 , LENGTH =128k
    ram(rwx)    : ORIGIN =0x20000000 , LENGTH =36k
}

/*Define Section*/
SECTIONS
{
    .text :
    {
        *(.isr_vector)
        *(.text)
        *(.rodata)
        _etext = .;
    }>flash
    
    .data :
    {
        _sdata = .;
        *(.data)
        _edata = .;
    }>ram AT> flash

    .bss :
    {
        _sbss = .;
        *(.bss)
        _ebss = .;
    }> ram
}
```
# Compiling asm file

Cause this project only have .s file , so we can produce .o file by arm-none-eabi-as.<br>

```bash
    arm-none-eabi-as main.s -g -o main.o
```

# Linking .o file to produce executable file

```bash
    arm-none-eabi-as -nostdlib -T stm32g070rbt.ld main.o -o main
```



# Startup openocd

```bash
    openocd -f /usr/share/openocd/scripts/interface/cmsis-dap.cfg -f /usr/share/openocd/scripts/target/stm32g0x.cfg
```

Here my target mcu is stm32g070rbt6,and debug tool is cmsis-dap.<br>

# Using arm-none-eabi-gdb

Debugging code with arm-none-eabi-gdb<br>

```bash

arm-none-eabi-gdb main
```

# Connecting hardware
Connecting hardware by localhost:3333

```gdb
target remote localhost:3333

```

```gdb
#reset chip
monitor rest
#halt chip
monitor chip
#load program
load
```
next step to debug !


