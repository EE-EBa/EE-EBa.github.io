---
title: Create a FreeRTOS manually based on cortex-m4  
date: 2025-03-19
categories: [FreeRTOS]
tags: [RTOS , makefile , GNU]
---

Usually we create a FreeRTOS with stm32cubeide , if we use stm32 mcu.
But this steps will hidden some detail. So I want to create a FreeRTOS 
project manually. It will help us understand more principle of FreeRTOS.

# 1. Import driver layer from mcu manufacturer
The layer includes startup file  ,link file  and other HAL library.

## Specific operation according to mcu of manufacturer
### for stm32
1. Copy the driver layer : STM32Cube/Repository/STM32Cube_FW_G4_V1.6.1/Drivers , it includes HAL and startup files.
2. Copy link file: STM32Cube/Repository/STM32Cube_FW_G4_V1.6.1/Projects/NUCLEO-G431RB/Templates/STM32CubeIDE/STM32G431RBTX_FLASH.ld
3. When you import this files, we need to select the target STMG4xx device in stm32g4xx.h , if we use cortex-m4 of stm32.
4. Add a stm32g4xx_hal_conf.h: STM32Cube/Repository/STM32Cube_FW_G4_V1.6.1/Projects/NUCLEO-G431RB/Templates/Inc/stm32g4xx_hal_conf.h 
# 2. Create BSP(Board Support Package) to initialize target MCU
when we create functions in bsp files. We need assert functions. So we need add qassert.c and qassert.h to implement assert functions. The qassert files come from QPC frame.
## for stm32
1. Configure system clock PLL etc. 

# 3. Import FreeRTOS Kernel

1. Import the FreeRTOSConfig.h : FreeRTOS-Kernel/examples/template_configuration/FreeRTOSConfig.h
2. Import portable : due to my compile tool chains is GCC, So the port directory is : 
  2.1 FreeRTOS-Kernel/portable/GCC/ARM_CM4F
  2.2 FreeRTOS-Kernel/portable/MemMang/heap_x.c
  2.3 FreeRTOS-Kernel/portable/Commin
3. Create default handler functions: interrupt.c/interrupt.h. Otherwise when we run program, it will jump into infinite loop.
```c 
/* interrupt.c */
#include "interrupt.h"

void TIM6_DAC_IRQHandler(void){

    HAL_TIM_IRQHandler(&htim6);
}
```
4. Rename stm32g4xx_hal_timebase_tim_template.c to stm32g4xx_hal_timebase_tim.c , The purpose of this operation is to allow us to set the clock source of HAL to a timer instead of system clock.
```c 
/* stm32g4xx_hal_timebase_tim.c */
/* Includes ------------------------------------------------------------------*/
#include "stm32g431xx.h"
#include "stm32g4xx_hal.h"
#include "stm32g4xx_hal_cortex.h"
#include "stm32g4xx_hal_rcc.h"
#include "stm32g4xx_hal_tim.h"

/** @addtogroup STM32G4xx_HAL_Driver
 * @{
 */

/** @addtogroup HAL_TimeBase
 * @{
 */

/* Private typedef -----------------------------------------------------------*/
/* Private define ------------------------------------------------------------*/
/* Private macro -------------------------------------------------------------*/
/* Private variables ---------------------------------------------------------*/
TIM_HandleTypeDef htim6;
/* Private function prototypes -----------------------------------------------*/
/* Private functions ---------------------------------------------------------*/

/**
 * @brief  This function configures the TIM6 as a time base source.
 *         The time source is configured  to have 1ms time base with a dedicated
 *         Tick interrupt priority.
 * @note   This function is called  automatically at the beginning of program
 * after reset by HAL_Init() or at any time when clock is configured, by
 * HAL_RCC_ClockConfig().
 * @param  TickPriority: Tick interrupt priority.
 * @retval HAL status
 */
HAL_StatusTypeDef HAL_InitTick(uint32_t TickPriority)
{
  RCC_ClkInitTypeDef    clkconfig;
  uint32_t              uwTimclock = 0;
  uint32_t              uwPrescalerValue = 0;
  uint32_t              pFLatency;
  HAL_StatusTypeDef     status;

  /* Enable TIM6 clock */
  __HAL_RCC_TIM6_CLK_ENABLE();

  /* Get clock configuration */
  HAL_RCC_GetClockConfig(&clkconfig, &pFLatency);

  /* Compute TIM6 clock */
  uwTimclock = HAL_RCC_GetPCLK1Freq();

  /* Compute the prescaler value to have TIM6 counter clock equal to 1MHz */
  uwPrescalerValue = (uint32_t) ((uwTimclock / 1000000U) - 1U);

  /* Initialize TIM6 */
  htim6.Instance = TIM6;

  /* Initialize TIMx peripheral as follow:

  + Period = [(TIM6CLK/1000) - 1]. to have a (1/1000) s time base.
  + Prescaler = (uwTimclock/1000000 - 1) to have a 1MHz counter clock.
  + ClockDivision = 0
  + Counter direction = Up
  */
  htim6.Init.Period = (1000000U / 1000U) - 1U;
  htim6.Init.Prescaler = uwPrescalerValue;
  htim6.Init.ClockDivision = 0;
  htim6.Init.CounterMode = TIM_COUNTERMODE_UP;

  status = HAL_TIM_Base_Init(&htim6);
  if (status == HAL_OK)
  {
    /* Start the TIM time Base generation in interrupt mode */
    status = HAL_TIM_Base_Start_IT(&htim6);
    if (status == HAL_OK)
    {
    /* Enable the TIM6 global Interrupt */
        HAL_NVIC_EnableIRQ(TIM6_DAC_IRQn);
      /* Configure the SysTick IRQ priority */
      if (TickPriority < (1UL << __NVIC_PRIO_BITS))
      {
        /* Configure the TIM IRQ priority */
        HAL_NVIC_SetPriority(TIM6_DAC_IRQn, TickPriority, 0U);
        uwTickPrio = TickPriority;
      }
      else
      {
        status = HAL_ERROR;
      }
    }
  }

 /* Return function status */
  return status;
}

/**
 * @brief  Suspend Tick increment.
 * @note   Disable the tick increment by disabling TIM6 update interrupt.
 * @param  None
 * @retval None
 */
void HAL_SuspendTick(void) {
  /* Disable TIM6 update interrupt */
  __HAL_TIM_DISABLE_IT(&htim6, TIM_IT_UPDATE);
}

/**
 * @brief  Resume Tick increment.
 * @note   Enable the tick increment by enabling TIM6 update interrupt.
 * @param  None
 * @retval None
 */
void HAL_ResumeTick(void) {
  /* Enable TIM6 update interrupt */
  __HAL_TIM_ENABLE_IT(&htim6, TIM_IT_UPDATE);
}
```
5. Create the functions vApplicationStackOverflowHook in middle layer . The purpose of this operation is to detect stack overflow.
```c 
/* freeRTOS_hooks.c */
#include "FreeRTOS.h"
#include "task.h"
#include "freeRTOS_hooks.h"
#include "stdio.h"

void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName)
{
    /* This function will be called if a task overflows its stack */
    printf("Stack overflow in task: %s\n", pcTaskName);
    
    /* Stop execution for debugging */
    for(;;);
}
```
6. Set macro configCPU_CLOCK_HZ as systemCoreClock in FreeRTOSConfig.h . The systemCoreClock is come from HAL. 
```c 
/* FreeRTOSConfig.h */
#if defined(__GNUC__)
#include <stdint.h>
extern uint32_t SystemCoreClock;
#endif

#define configCPU_CLOCK_HZ    ( SystemCoreClock )

```
7. Set macros ,vPortSVCHandler/xPortPendSVHandler/xPortSysTickHandler in FreeRTOSConfig.h . Definitions that map the FreeRTOS port interrupt handlers to their CMSIS
```c 
/* FreeRTOSConfig.h */
#define vPortSVCHandler SVC_Handler
#define xPortPendSVHandler PendSV_Handler
#define xPortSysTickHandler SysTick_Handler
```
8. Set macros about system call priority and  interrupt priority.
```c 
/* FreeRTOSConfig.h */
/* These values are specific to Cortex-M4 (STM32G431): */
#define configPRIO_BITS                         4 /* 16 priority levels on STM32G4 */
#define configLIBRARY_LOWEST_INTERRUPT_PRIORITY 15
#define configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY 5
#define configKERNEL_INTERRUPT_PRIORITY         ( configLIBRARY_LOWEST_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )
#define configMAX_SYSCALL_INTERRUPT_PRIORITY    ( configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )
```


