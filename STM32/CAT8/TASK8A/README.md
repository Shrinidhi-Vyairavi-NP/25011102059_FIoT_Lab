# Continuous Lab Evaluation-8

**EXP:** Continuous Lab Evaluation-8  
**DATE:** 18/09/2026  
**NAME OF EXP:** STM32 GPIO Bring-Up and Switch-Controlled LED  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Development Board (Blue Pill) + Push-Button + LED  
**TASK NO:** 8A  

---

### Circuit Diagram
![Circuit Diagram](../CAT8.png)

---

### Core Logic

* **1. Switch Polling & State Mirroring**
  Continuously reads the input logic level on `PA0` using `HAL_GPIO_ReadPin()` inside the main `while (1)` loop and directly mirrors that state onto `PA5` using `HAL_GPIO_WritePin()`, keeping the LED ON while the active-LOW switch is held down and OFF when released.
  ```c
  if (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == GPIO_PIN_RESET)
  {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
  }
  else
  {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
  }
  HAL_Delay(10);
  ```

---

### Inference
GPIO input and output peripherals were successfully configured on the STM32 Blue Pill using STM32CubeMX and Keil MDK-ARM, verifying direct polling-based digital control where the LED output tracks the real-time logic state of the push-button.