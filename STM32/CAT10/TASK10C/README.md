# Continuous Lab Evaluation - 10

**EXP:** Continuous Lab Evaluation-10 
**DATE:** 14/11/2026  
**NAME OF EXP:** Connect Wemos to Wi-Fi  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 BluePill (STM32F103C8T6, 72 MHz HSE) + Wemos D1 Mini (ESP8266) + USB-to-TTL Converter  
**TASK NO:** 10C  

---

### Circuit Diagram
![Circuit Diagram](../CAT10.png)

---

### Core Logic

* **1. Station Mode Configuration & Access Point Authentication**
  Configures the ESP8266 as a Wi-Fi client (`CWMODE=1`) and authenticates with the target access point (`CWJAP`) using an extended `12000 ms` timeout to accommodate DHCP handshake delays.
  ```c
  send_cmds("AT\r\n", 1000);
  HAL_Delay(1000);

  send_cmds("AT+CWMODE=1\r\n", 1000);
  HAL_Delay(1000);

  send_cmds("AT+CWJAP=\"SNUC\",\"snu12345\"\r\n", 12000);
  HAL_Delay(1000);
  ```

* **2. Network Status & IP Verification**
  Queries the Wemos module using `AT+CIFSR` once connected to retrieve and display the assigned Station IPv4 address (`STAIP`) and MAC address (`STAMAC`) on the PC terminal.
  ```c
  send_cmds("AT+CIFSR\r\n", 3000);
  HAL_Delay(1000);

  char *summary = "\r\n--- VERIFICATION ---\r\n"
                  "WIFI MODE   : STATION\r\n"
                  "WIFI STATUS : CONNECTED\r\n"
                  "--------------------\r\n";
  HAL_UART_Transmit(&huart3, (uint8_t *)summary, strlen(summary), HAL_MAX_DELAY);
  ```

---

### Inference
The STM32 firmware successfully automated the Wi-Fi station connection workflow on the Wemos D1 Mini using sequential AT commands. Providing an adequate polling window during `AT+CWJAP` allowed the ESP8266 to complete association and DHCP negotiation (`WIFI CONNECTED` and `WIFI GOT IP`), enabling `AT+CIFSR` to retrieve and report a valid local IP address on the serial monitor.