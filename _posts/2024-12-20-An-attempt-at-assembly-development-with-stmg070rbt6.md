---
title: An attempt at assembly development with an STM32G070RBT6
categories: [Embedded , openocd]
tags: [openocd , cortex , debug]

---

# download STM32CubeMx
get resource and example related to STM32GO70RBT6 

# startup file (*.s) 
* [startup reference](https://www.emoe.xyz/stm32-boot-modeboot-filelinkerscript-analyze/#33_ROMRAM)
* [arm assembly](https://blog.csdn.net/weixin_42328389/article/details/121855164)
 
# some  assembly instruction of cortex-m0plus can't correct use 
I was getting to ready use "orr" instruction. Here's the code form I wrote.

```asm
.syntax unfied
.cpu cortex-m0plus
    .text
    .global ASM_Function
    .thumb
ASM_Function:
    ldr     r0 , =(0x50000800 + 0x0c)
    ldr     r1 , [r0]
    orrs    r1 , r1 , #(1 << 3)
    str     r1 , [r0]
```

whatever I tried to , after build , debuge message is "Error: cannot honor width suffix -- `orrs r1,r1,#(1<<3)'"<br>
I loock for "ARMv6-M Architecture Reference Manual" , I discover orr instruction can't support immediate operation in armv6.<br>
It only support register operation.

Here's the correct code form .

```asm
.syntax unfied
.cpu cortex-m0plus
    .text
    .global ASM_Function
    .thumb
ASM_Function:
    ldr     r0 , =(0x50000800 + 0x0c)
    ldr     r1 , [r0]
    ldr     r2 , =(1 << 3)
    orrs    r1 , r1 , r2
    str     r1 , [r0]
```

# openocd can't run(on ubuntu)
Here's the error message <br>

```
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Error: The specified debug interface was not found (cmsis-dap)
The following debug adapters are available:
1: buspirate
```

solved method :<br>
1. uninstall openocd(remove install folder,/usr/local/share/openocd , /usr/local/bin/openocd)
2. install some dependency
3. install openocd
