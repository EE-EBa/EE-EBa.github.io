---
title: How to diy a dap debuger
date: 2024-12-20
categories: [Embedded]
tags: [dap]     # TAG names should always be lowercase



# 以下默认false
math: true
mermaid: true

# 以下默认ture
toc: false
comments: false
---

# knowledge
1. dap-link is based on and replaces cmsis-dap. 
2. cmsis-dap specifies both jtag and swd commands,dap-link provides support for both.
3. dap-link or cmsis-dap debuger is also a minimal mcu circuit. The mcu can be stm32 or nxp or riscv.
4. dap-link's firmware must be programmed by stlink/dap-link or other debuger.
5. According to dap-link document,dap-link's firmware include two section. One is bootloader , second is firmware code.


# ref
1. [DAP-LINK](https://github.com/ARMmbed/DAPLink/tree/main)
3. [DAP_LINK下载器固件编译下载过程](https://blog.csdn.net/weixin_45829708/article/details/124359707)
4. [Build Your Own Low Cost CMSIS-DAP Debug Unit](https://ravikiranb.com/projects/cmsis-dap-debug/)
5. [DAPLink and the USB interface](https://tech.microbit.org/software/daplink-interface/)
