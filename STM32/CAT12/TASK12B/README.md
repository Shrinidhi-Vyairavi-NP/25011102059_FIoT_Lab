# Continuous Lab Evaluation-12

**EXP:** Continuous Lab Evaluation-12  
**DATE:** 14/10/2026  
**NAME OF EXP:** On-board Switch Control with State Remembrance  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Nucleo Development Board (STM32F401RE Cortex-M4)  
**TASK NO:** 2  

---

### Circuit Diagram
![Circuit Diagram](../CAT12.png)

---

### Core Logic

* **1. Active-Low Switch Polling & Software Debouncing**
  Polls the on-board blue push button (`B1` on `PC13`) for an active-low press (`GPIO_PIN_RESET`) and applies a 50 ms delay followed by a second verification read to filter mechanical contact bounce.
  ```c
  if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
  {
    HAL_Delay(50);
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
    {
      HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
      while (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET);
    }
  }
  ```

* **2. State Remembrance & Single-Trigger Release Lockout**
  Flips the `PA5` output data register once per valid press so the LED retains its toggled state after switch release, while the blocking `while` loop prevents repeated toggling if the button remains held down.

---

### Inference
Implementing a 50 ms software debounce check and release lockout eliminated false triggers caused by mechanical switch bounce on `PC13`. Each discrete press of the on-board push button cleanly toggled the LED (`LD2`) once and preserved its logic state upon release.