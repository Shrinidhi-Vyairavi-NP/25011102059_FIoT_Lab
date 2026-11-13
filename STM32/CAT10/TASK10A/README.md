# Continuous Lab Evaluation - 10

**EXP:** Continuous Lab Evaluation-10 
**DATE:** 14/11/2026  
**NAME OF EXP:** Establish STM32 ↔ Wemos UART Communication  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 BluePill (STM32F103C8T6, 72 MHz HSE) + Wemos D1 Mini (ESP8266) + USB-to-TTL Converter  
**TASK NO:** 10A

---

### Circuit Diagram
![Circuit Diagram](../CAT10.png)

---

### Core Logic

* **1. Serial Command Transmission & Terminal Logging**
  Formats the outgoing `AT\r\n` test string, logs the command to the PC serial terminal (`USART3`), and transmits it to the Wemos D1 Mini over `USART1` at 115200 baud.
  ```c
  char at_cmd[] = "AT\r\n";
  snprintf(send, sizeof(send), "SENDING\r\n : %s", at_cmd);
  HAL_UART_Transmit(&huart3, (uint8_t *)send, strlen(send), HAL_MAX_DELAY);
  HAL_UART_Transmit(&huart1, (uint8_t *)at_cmd, strlen(at_cmd), HAL_MAX_DELAY);
  ```

* **2. Byte-by-Byte Response Polling**
  Continuously polls `USART1` within a non-blocking tick window (`1000 ms`) to capture incoming characters from the Wemos and immediately forwards each received byte (`OK\r\n`) to PuTTY over `USART3`.
  ```c
  uint32_t overall_start = HAL_GetTick();
  while ((HAL_GetTick() - overall_start) < 1000) {
      if (HAL_UART_Receive(&huart1, &rx_byte, 1, 10) == HAL_OK) {
          HAL_UART_Transmit(&huart3, &rx_byte, 1, HAL_MAX_DELAY);
      }
  }
  ```

---

### Inference
Bidirectional UART serial communication between the STM32F103C8T6 and the Wemos D1 Mini was successfully verified at 115200 baud. Transmitting `AT\r\n` with proper CR/LF line endings yielded an `OK` acknowledgment on the PC terminal, confirming accurate baud rate synchronization, common ground referencing, and hardware cross-wiring before attempting Wi-Fi configuration.