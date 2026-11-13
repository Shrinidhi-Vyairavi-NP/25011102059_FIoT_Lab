# Continuous Lab Evaluation-8 

**EXP:** Continuous Lab Evaluation-8  
**DATE:** 18/09/2026  
**NAME OF EXP:** STM32 GPIO Bring-Up and Switch-Controlled LED  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Development Board (Blue Pill) + Push-Button + LED  
**TASK NO:** 8B  

---

### Circuit Diagram
![Circuit Diagram](../CAT8.png)

---

### Core Logic

* **1. Software Debouncing & Edge-Triggered Toggle**
  Filters mechanical contact bounce using a `20 ms` verification delay (`HAL_Delay(20)`) upon detecting an active-LOW press, toggles the LED output via `HAL_GPIO_TogglePin()`, and blocks on a `while` release check so a single press-and-release cycle produces exactly one toggle.
  ```c
  if (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == GPIO_PIN_RESET)
  {
      HAL_Delay(20);
      if (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == GPIO_PIN_RESET)
      {
          HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
          while (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == GPIO_PIN_RESET);
          HAL_Delay(20);
      }
  }
  ```

---

### Inference
Implementing a 20 ms software debounce window combined with a blocking release check effectively eliminated mechanical switch chatter, ensuring reliable one-to-one state toggling of the LED for each distinct push-button press.