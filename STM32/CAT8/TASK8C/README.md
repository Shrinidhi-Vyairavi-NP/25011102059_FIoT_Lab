# Continuous Lab Evaluation-8

**EXP:** Continuous Lab Evaluation-8  
**DATE:** 18/09/2026  
**NAME OF EXP:** STM32 GPIO Bring-Up and Switch-Controlled LED  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Development Board (Blue Pill) + Push-Button + LED  
**TASK NO:** 8C  

---

### Circuit Diagram
![Circuit Diagram](../CAT8.png)

---

### Core Logic

* **1. Non-Blocking Press Duration Measurement**
  Captures timestamps with `HAL_GetTick()` on button press and release edges to calculate total hold time (`press_duration`) without blocking system execution, allowing active LED blinking to continue seamlessly while the button is held down.
  ```c
  uint32_t press_start = HAL_GetTick();
  while (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == GPIO_PIN_RESET)
  {
      if (is_blinking && (HAL_GetTick() - last_blink_time >= 200))
      {
          HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
          last_blink_time = HAL_GetTick();
      }
  }
  uint32_t press_duration = HAL_GetTick() - press_start;
  ```

* **2. Event Classification & State Machine Control**
  Classifies presses `< 1000 ms` as short presses (stopping blinking mode if active, otherwise toggling the LED) and presses `>= 1000 ms` as long presses (enabling continuous non-blocking `200 ms` LED blinking via `is_blinking`).
  ```c
  if (press_duration >= 1000)
  {
      is_blinking = 1;
  }
  else
  {
      if (is_blinking)
      {
          is_blinking = 0;
          HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
      }
      else
      {
          HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
      }
  }
  ```

---

### Inference
Using the non-blocking SysTick millisecond counter (`HAL_GetTick()`) enabled accurate differentiation between short (`< 1 s`) and long (`>= 1 s`) push-button events without halting firmware execution, supporting smooth state transitions between single LED toggling and continuous blinking modes.