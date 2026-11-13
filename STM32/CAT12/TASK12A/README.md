# Continuous Lab Evaluation-12

**EXP:** Continuous Lab Evaluation-12  
**DATE:** 14/10/2026  
**NAME OF EXP:** On-board LED Bring-Up  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Nucleo Development Board (STM32F401RE Cortex-M4)  
**TASK NO:** 1  

---

### Circuit Diagram
![Circuit Diagram](../CAT12.png)

---

### Core Logic

* **1. Periodic GPIO Output Toggling & Fixed Delay**
  Continuously inverts the logic state of the on-board green user LED (`LD2` on `PA5`) inside the infinite `while (1)` loop and introduces a deterministic 500 ms SysTick delay between transitions to achieve a 1 Hz blink cycle.
  ```c
  while (1)
  {
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(500);
  }
  ```

---

### Inference
The STM32F401RE GPIO push-pull output configuration and 84 MHz HSE bypass clock tree were verified via hardware bring-up. Using `HAL_GPIO_TogglePin()` alongside SysTick-driven `HAL_Delay()` produced a consistent, drift-free 500 ms blink interval on the on-board user LED (`LD2`).