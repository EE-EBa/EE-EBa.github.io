---
title: Add a SystemView configuration into a FreeRTOS project 
date: 2025-03-26
categories: [FreeRTOS]
tags: [ openocd, debug,RTOS ,GNU,makefile]
---

# 1. Import generic configuration

It is officially recommended to import all files from SystemView's Target-src.

# 2. Delete  other rtos folder in ./ThirdParty/SEGGER/Target-src/Sample/

Delete all folders , just leave FreeRTOS11

# 3. Initializing SystemView

The system information are sent by the application. This information can be configured via
defines in SEGGER_SYSVIEW_Config_[SYSTEM].c. Add a call to SEGGER_SYSVIEW_Conf() in
the main function to initialize SystemView.

```c 
  int main(void) {
  HAL_Init();

  BSP_Init();
  
  SEGGER_SYSVIEW_Conf();
  SEGGER_SYSVIEW_Start();

  /* Create tasks(threads) */
  status = xTaskCreate(Task1_Handler, "Task-1", 200, "hello world from task-1",
                       2, &Task1_TCB);
  Q_ASSERT(status == pdPASS);

  status = xTaskCreate(Task2_Handler, "Task-2", 200, "hello world from task-2",
                       2, &Task2_TCB);
  Q_ASSERT(status == pdPASS);

  /* Start the FreeRTOS scheduler */
  vTaskStartScheduler();

  printf("if the program comes here , represent the launch of the scheduler "
         "has failed due to insufficient ram!\n");
  for (;;)
    ;
  return 0;
}
```

As long as the SystemView Application is not connected, and SEGGER_SYSVIEW_Start() is not called, the application will not generate SystemView events. When SystemView is connected or SEGGER_SYSVIEW_Start() is called it will activate recording SystemView events.

For single-shot recording SEGGER_SYSVIEW_Start() must be called in the application to activate recording SystemView events.Events are recorded until the SystemView buffer is filled or SEGGER_SYSVIEW_Stop() is called.

# 4. FreeRTOSConfig.h Setting(from course)

1. SEGGER_SYSVIEW_FreeRTOS.h header must be included at the end of FreeRTOSConfig.h or above every include of FreeRTOS.h. It defines the trace macro to create SYSTEMVIEW event.
2. In FreeRTOSConfig.h include the below macros

```c 
#define INCLUDE_xTaskGetIdleTaskHandle  1 
#define INCLUDE_pxTaskGetStackStart     1 /* But this macro is not include in FreeRTOSv11 */
```

# 5. Specify target mcu in SEGGER_SYSVIEW_ConfDefault.h

```c 
  #ifndef   SEGGER_SYSVIEW_CORE
    #define SEGGER_SYSVIEW_CORE SEGGER_SYSVIEW_CORE_CM3 
  #endif
```

# 6. Do SystemView buffer size configuration in SEGGER_SYSVIEW_ConfDefault.h(SEGGER_SYSVIEW_RTT_BUFFER_SIZE)

```c 

/*********************************************************************
*
*       Define: SEGGER_SYSVIEW_RTT_BUFFER_SIZE
*
*  Description
*    Number of bytes that SystemView uses for the RTT buffer.
*  Default
*    1024
*/
#ifndef   SEGGER_SYSVIEW_RTT_BUFFER_SIZE
  #define SEGGER_SYSVIEW_RTT_BUFFER_SIZE          (4 * 1024)
#endif

```

# 6. Enable the  cycle counter

This is required to maintain the time stamp information of application events. SystemView will use the cycle counter register value to keep the time stamp information of the events.

## for ARM Cortex M3/M4
DWT_CYCCNT register of ARM Cortex M3/M4 processor stores the number of clock cycles that happened after the processor's reset. By default , this register is disabled,Because M0 and M0+ is not support it.


```c 

#define DWT_CTRL (*(volatile uint32_t *)0xe0001000)

int main(void) {
  HAL_Init();

  BSP_Init();
  
  DWT_CTRL |= (1 << 0);
  }
```


# 7. Compile and debug

If using Cortex-m4 , we need to write bellow code:

```c 
int main(void) {
  HAL_Init();

  BSP_Init();
  NVIC_SetPriorityGrouping(0); /* FreeRTOS Requirement: FreeRTOS requires a specific configuration 
                                where all bits are used for preemption priority (and none for subpriority). 
                                This is achieved by setting the priority grouping to 0 via NVIC_SetPriorityGrouping(0) */


  DWT_CTRL |= (1 << 0);
  SEGGER_SYSVIEW_Conf();
  SEGGER_SYSVIEW_Start();
 
 }
```


# 8. Generate .SVDat file

My debug tools is st-link+openocd, so for simplify, make SystemView woke in single-shot recode. So .SVDat file is necessary.
* Hit run and the pause after couple of seconds.
* Write down the code  bellow 

```gdb
# .gdbint

define memory_log
	set logging enable on
	dump binary memory 001.SVDat _SEGGER_RTT.aUp[1].pBuffer _SEGGER_RTT.aUp[1].pBuffer+_SEGGER_RTT.aUp[1].WrOff
	set logging off
end
```

* Add a description at the top of 001.SVDat,otherwise SystemView can not recognise this file.

```
;
; Version     SEGGER SystemView V2.38
; RecordTime  06 Jul 2016 17:24:53
; Author      Johannes
; Title       embOS/IP Webserver
; Description 
;


```

