---
title: 
date: 2025-03-19
categories: [FreeRTOS,archlinux,Embedded]
tags: [assmebly, nvim, openocd, debug,RTOS ,GNU,makefile, Linux , iwctl , git , stow , dotfiles]
---


#  About debugging with SystemView 

* issue1: Jump HardFault function

I create a task what is contain a variable,and I wanna print message on SystemView.

```c 
status = xTaskCreate(Task1_Handler, "Task-1", 400, "hello world from task-1",
                       2, &Task1_TCB);

static void Task1_Handler(void *parameters) {
  char msg[100];
  for (;;) {
    snprintf(msg, 100, "%s\n", (char *)parameters);
    SEGGER_SYSVIEW_PrintfTarget(msg);
    taskYIELD();
  }
}

```

When the stack's depth is 200, i will jump the HardFault function. 
This is due to the task's dynamic memory run run out. Increasing the stack's depth at 400. It is solved.

* What is the state of task?
It includes **contents of the processor core register** + **stack contents**.


* A abnormal issue in  xPortPendSVHandler 

```c 
/* declare a pointer  */
portDONT_DISCARD PRIVILEGED_DATA TCB_t * volatile pxCurrentTCB = NULL;

pxCurrentTCB = &Task1_TCB; /* stand for assigning the address of Task1_TCB to pxCurrentTCB ,not changge the address of pxCurrentTCB*/
```


```asm 
ldr r3, =pxCurrentTCB
```

in assmebly , the pxCurrentTCB stand for the **address of the pointer**, not a value of the pointer


