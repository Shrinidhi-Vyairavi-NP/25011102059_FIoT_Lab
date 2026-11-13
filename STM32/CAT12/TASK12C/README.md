# Continuous Lab Evaluation-12

**EXP:** Continuous Lab Evaluation-12  
**DATE:** 14/10/2026  
**NAME OF EXP:** Short-Press and Long-Press Detection  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Nucleo Development Board (STM32F401RE Cortex-M4)  
**TASK NO:** 3  

---

### Circuit Diagram
![Circuit Diagram](../CAT12.png)

---

### Core Logic

* **1. SysTick Press Duration Measurement**
  Captures the initial `HAL_GetTick()` timestamp upon detecting a debounced active-low press on `PC13` and computes total hold duration in milliseconds once the button is released.
  ```c
  uint32_t press_start = HAL_GetTick();
  while (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
  {
    if (blinking_mode && (HAL_GetTick() - last_blink_time >= 200))
    {
      HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
      last_blink_time = HAL_GetTick();
    }
  }
  uint32_t press_duration = HAL_GetTick() - press_start;
  ```

* **2. Duration-Based Mode Switching & Non-Blocking Blinking**
  Evaluates the measured hold time against a 1000 ms threshold—activating continuous 200 ms non-blocking blinking on a long press ($\ge 1000\text{ ms}$) and either stopping blinking or toggling the static LED state on a short press ($< 1000\text{ ms}$).
  ```c
  if (press_duration >= 1000)
    blinking_mode = 1;
  else
  {
    if (blinking_mode) blinking_mode = 0;
    else HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
  }

  if (blinking_mode && (HAL_GetTick() - last_blink_time >= 200))
  {
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    last_blink_time = HAL_GetTick();
  }
  ```

---

### Inference
Using `HAL_GetTick()` timestamps enabled accurate differentiation between short presses ($< 1\text{ s}$) and long presses ($\ge 1\text{ s}$) without requiring an auxiliary hardware timer. Non-blocking interval checks maintained responsive button polling while transitioning seamlessly between static LED toggling and continuous blinking modes.