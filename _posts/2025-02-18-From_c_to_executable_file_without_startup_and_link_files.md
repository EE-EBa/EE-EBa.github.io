---
title: Debug mcu with commands ,no ld file and startup file.
date: 2025-02-18
categories: [Embedded]
tags: [debug]
---


# 背景
为了进一步理解嵌入式c语言在mcu中的调试过程,本文采用纯命令行,不带启动文件与链接文件进行c程序调试.以验证某些c语言在mcu中的功能.

调试程序如下:
```c
/* test.c */
#include "stdint.h"
#include <stdio.h>
/* base class definition  */
typedef struct {
  int16_t i16_x;
  int16_t i16_y;
  char i8_type;
} Shape;

/* subclass definition */
typedef struct {
  Shape super;

  int16_t i16_width;
  int16_t i16_height;
} Rectangle;

typedef struct {
  Shape super;

  int16_t i16_radius;
} Circle;

/* method of class  */
void Shape_draw(Shape const *const me) { printf("Drawing a generic shape\n "); }

void Rectangle_draw(Rectangle const *const me) {
  printf("Drawing a rectangle shape\n ");
}

void Circle_draw(Circle const *const me) {
  printf("Drawing a circle shape\n ");
}

/* constructor of class */
void Shape_ctor(Shape *const me, int16_t i16_x0, int16_t i16_y0) {
  me->i16_x = i16_x0;
  me->i16_y = i16_y0;
  me->i8_type = 'S';
}

void Rectangle_ctor(Rectangle *const me, int16_t i16_x0, int16_t i16_y0,
                     int16_t i16_w0, int16_t i16_h0) {
  me->super.i16_x = i16_x0;
  me->super.i16_y = i16_y0;
  me->super.i8_type = 'R';
  me->i16_height = i16_h0;
  me->i16_width = i16_w0;
}

void Circle_ctor(Circle *const me, int16_t i16_x0, int16_t i16_y0,
                 int16_t i16_r0) {
  me->super.i16_x = i16_x0;
  me->super.i16_y = i16_y0;
  me->super.i8_type = 'C';
  me->i16_radius = i16_r0;
}


/* generic function of drawing */
void Draw_Shape(void const * const me){
    Shape * base = (Shape *)me;
    switch(base->i8_type){
        case 'R':
            Rectangle_draw((Rectangle *)me);
        case 'S':
            Shape_draw((Shape * )me);
        case 'C':
            Circle_draw((Circle *)me);
    }
}

Shape s1;
Rectangle r1;
Circle c1;


int main(void){

Shape_ctor(&s1, 1, 2);
Rectangle_ctor(&r1, 3, 4, 5, 6);
Circle_ctor(&c1, 7, 8, 9);

/* create array of shapes */
void * shapes[]={
    &s1,
    &r1,
    &c1
};

/* call function */
for(int i = 0;i<2;i++){
    printf("\nShape %d:\n",i+1);
    Draw_Shape(shapes[i]);
}

return 0;


}


```

上述程序没有关于外设的操作,因此可以直接运行在mcu上,无需初始话时钟和其他外设.

# 手动编译步骤
1. 生成不可执行的.o文件
```shell

arm-none-eabi-gcc -mcpu=cortex-m4 -mthumb -I/usr/arm-none-eabi/include -c -g test.c -o test.o 
```
需要注意的是必须指明烧录的mcu架构还有指明使用的指令集,这里使用的是mthumb指令集.

2. 将.o文件链接成可以执行烧录的文件
```shell
arm-none-eabi-gcc -mcpu=cortex-m4 -mthumb -Ttext=0x08000000  -o test.elf test.o -specs=rdimon.specs -lc -lgcc -lrdimon -e main

```
注意:在链接阶段同样需要指明架构和指令集,还有代码保存的位置,这里代码保存在rom里且地址从0x08000000开始,由于test.c代码中含有printf这种标准库里的函数,因此需要使用-specs=rdimon.specs -lc -lgcc -lrdimon,同样还需要指明函数的入口地址,这里设置在main函数. 这里arm-none-eabi-gcc与arm-none-eabi-ld具有相同的功能,但是二者不能完全替换.感兴趣的可以替换试试.

# debug注意事项
在debug时,由于指令直接从main函数开始运行,没有对堆栈起始地址进行设置,因此如果直接运行程序的话, 程序会跑飞的. 当进入调试阶段后,首先做的是初始化sp寄存器,这里将sp寄存器初始化为ram的最高位.在stm32g431rbt6中stack=0x08000000 + 32K.因此可以在gdb中输入以下命令:
```shell
gdb: set $sp=0x08000000 + 32*1024
```
之所以进入debug后首先de设置sp的原因是,由于入口地址是main函数,进入main函数的第一件事情就是压栈,如果不设置好sp寄存器的值,那么程序肯定会跑飞的.

至此可以顺利的进行程序调试了.
