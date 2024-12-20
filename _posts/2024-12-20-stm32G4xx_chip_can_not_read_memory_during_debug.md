---
title:  A debugging issue:"Unable to read memory" when connecting to openocd
categories: [Embedded , issue]
tags: [openocd , debug]
---


## Background
When I debug following code, some wrong is happened.

```c 
int main(void)
{

  /*Enable GPIOA clock  through configuring RCC*/
  *(unsigned int*)(0x40021000 + 0x4c) =  0x1;

  /*Enable GPIOA3 as output through configuring GPIOx_MODER */
  *(unsigned int*)(0x48000000 ) =(0x00000040) ;

    /*Turn on led through configuring CPIOx_PUPDR*/
  *(unsigned int*)(0x48000000 + 0x0c)= 0x00000080;
    /*Turn off led through configuring CPIOx_PUPDR*/
  *(unsigned int*)(0x48000000 + 0x0c)= 0x00000040;

  

    return 0;

}
```



Here is error message with stlink:



jtag status contains invalid mode value - communication failure
Previous state query failed, trying to reconnect
jtag status contains invalid mode value - communication failure


 or if you are using jlink,the error is can not read memory.

**Actually , the root error is stm32 chip was locked. Generally we can set flash option register(FLASH_OPTR)'s RDP byte to solve the error.**


# Solved steps

1. Connect chip under reset.

```sh
st-info --probe  --connect-under-reset #note push reset button on your board.
```

2. Setting FLASH_OPTR's RDP is level 0 to unlock chip

```sh 
st-flash erase
```

3. Correcting  code 

The mistake is change GPIOA13 and GPIOA14. They are debug bit, Do not change. 

```c
int main(void)
{

  /*Enable GPIOA clock  through configuring RCC*/
  *(unsigned int*)(0x40021000 + 0x4c) =  0x1;

  /*Enable GPIOA3 as output through configuring GPIOx_MODER */
  *(unsigned int*)(0x48000000 ) =(*(unsigned int*)(0x48000000 )) | (0x00000040) ;

    /*Turn on led through configuring CPIOx_PUPDR*/
  *(unsigned int*)(0x48000000 + 0x0c)= 0x00000080;
    /*Turn off led through configuring CPIOx_PUPDR*/
  *(unsigned int*)(0x48000000 + 0x0c)= 0x00000040;

  

    return 0;

}

```
