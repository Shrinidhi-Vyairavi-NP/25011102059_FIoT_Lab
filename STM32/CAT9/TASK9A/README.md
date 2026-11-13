# Continuous Lab Evaluation-9

**EXP:** Continuous Lab Evaluation-9  
**DATE:** 28/09/2026  
**NAME OF EXP:** STM32 UART Command Controlled LED  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Development Board (STM32F103C8T6 Blue Pill) + USB-to-TTL Converter  
**TASK NO:** 9A  

---

### Circuit Diagram
![Circuit Diagram](../CAT9.png)

---

### Core Logic

* **1. UART Peripheral Initialization**
  Configures `USART1` in asynchronous mode (`115200 baud`, 8 data bits, no parity, 1 stop bit) via STM32CubeMX and initializes it using `HAL_UART_Init()`.
  ```c
  huart1.Instance = USART1;
  huart1.Init.BaudRate = 115200;
  huart1.Init.WordLength = UART_WORDLENGTH_8B;
  huart1.Init.StopBits = UART_STOPBITS_1;
  huart1.Init.Parity = UART_PARITY_NONE;
  HAL_UART_Init(&huart1);
  ```

* **2. Non-Blocking Character Reception & Echo**
  Polls `USART1` with a short 10 ms timeout so incoming bytes from the PC terminal are read and immediately echoed back without stalling the main loop.
  ```c
  if (HAL_UART_Receive(&huart1, &rx_data, 1, 10) == HAL_OK) {
      HAL_UART_Transmit(&huart1, &rx_data, 1, 100);
  }
  ```

* **3. Periodic Character Transmission**
  Uses `HAL_GetTick()` instead of blocking delays to transmit a single character (`'A'`) repeatedly every 1000 ms to the PC terminal.
  ```c
  if (HAL_GetTick() - last_tx_time >= 1000) {
      HAL_UART_Transmit(&huart1, &periodic_char, 1, 100);
      last_tx_time = HAL_GetTick();
  }
  ```

---

### Inference
Full-duplex asynchronous serial communication was successfully established between the STM32 Blue Pill and the PC terminal over USART1 using `HAL_UART_Transmit()` and `HAL_UART_Receive()`. Utilizing non-blocking timeouts alongside `HAL_GetTick()` enabled simultaneous periodic character transmission and real-time keystroke echoing without data loss.