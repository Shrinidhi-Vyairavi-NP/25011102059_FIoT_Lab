# Continuous Lab Evaluation - 10

**EXP:** Continuous Lab Evaluation-10  
**DATE:** 14/11/2026  
**NAME OF EXP:** Explore the ESP8266 AT Command Interface  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 BluePill (STM32F103C8T6, 72 MHz HSE) + Wemos D1 Mini (ESP8266) + USB-to-TTL Converter  
**TASK NO:** 10B 

---

### Circuit Diagram
![Circuit Diagram](../CAT10.png)

---

### Core Logic

* **1. Modular Command Dispatcher (`send_cmds`)**
  Encapsulates UART transmission and timed response streaming so individual AT commands can be tested sequentially with custom timeout windows suited to their execution duration.
  ```c
  void send_cmds(char *cmd, uint32_t total_wait_time) {
      char send[100];
      snprintf(send, sizeof(send), "SENDING\r\n : %s", cmd);
      HAL_UART_Transmit(&huart3, (uint8_t *)send, strlen(send), HAL_MAX_DELAY);
      HAL_UART_Transmit(&huart1, (uint8_t *)cmd, strlen(cmd), HAL_MAX_DELAY);

      uint8_t rx_byte;
      uint32_t overall_start = HAL_GetTick();
      while ((HAL_GetTick() - overall_start) < total_wait_time) {
          if (HAL_UART_Receive(&huart1, &rx_byte, 1, 10) == HAL_OK) {
              HAL_UART_Transmit(&huart3, &rx_byte, 1, HAL_MAX_DELAY);
          }
      }
  }
  ```

* **2. AT Command Investigation Sequence**
  Queries firmware version details, sets the Wi-Fi operating mode to Station (`1`), scans nearby access points, and checks local IP/MAC assignment parameters.
  ```c
  send_cmds("AT\r\n", 1000);          // Check modem response
  send_cmds("AT+GMR\r\n", 2000);      // Check firmware/SDK version
  send_cmds("AT+CWMODE=1\r\n", 1000); // Set Wi-Fi station mode
  send_cmds("AT+CWLAP\r\n", 5000);    // Scan available networks
  send_cmds("AT+CIFSR\r\n", 3000);    // Obtain network IP/MAC info
  ```

---

### Inference
Testing the ESP8266 AT command set via the STM32 UART bridge confirmed how different modem operations require tailored timeout durations. Basic status and mode commands (`AT`, `AT+CWMODE=1`) respond within milliseconds, whereas network discovery (`AT+CWLAP`) streams multi-line AP data over several seconds, requiring an extended polling window to prevent response truncation.