# Continuous Lab Evaluation-11

**EXP:** Continuous Lab Evaluation-11  
**DATE:** 12/10/2026  
**NAME OF EXP:** HTTP Server - STM32–Wemos Wi-Fi Modem Using AT Commands  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Development Board + WemosD1 WiFi Module  
**TASK NO:** 11  

---

### Circuit Diagram
![Circuit Diagram](../CAT11.png)

---

### Core Logic

* **1. Wi-Fi Access Point Connection**
  Configures the ESP8266 to operate in Station mode and connects to the specified local network using the SSID and password.
  ```c
  send_cmds("AT+CWMODE=1\r\n", 1000);
  send_cmds("AT+CWJAP=\"SNUC\", \"snu12345\"\r\n", 12000);
  ```

* **2. TCP Server Initialization**
  Enables multiple concurrent connections (`CIPMUX=1`) which is required for server mode, and then starts a TCP server listening on the standard HTTP port 80.
  ```c
  send_cmds("AT+CIPMUX=1\r\n", 1000);
  send_cmds("AT+CIPSERVER=1,80\r\n", 3000);
  ```

---

### Inference
The Wemos D1/ESP8266 module was successfully configured via UART AT commands from the STM32 to connect to a local Wi-Fi network and establish a TCP server on port 80, preparing it to listen for incoming network requests.