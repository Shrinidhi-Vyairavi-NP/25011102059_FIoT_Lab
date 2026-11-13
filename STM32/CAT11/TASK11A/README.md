# Continuous Lab Evaluation-11

**EXP:** Continuous Lab Evaluation-11  
**DATE:** 12/10/2026  
**NAME OF EXP:** HTTP Server - STM32–Wemos Wi-Fi Modem Using AT Commands  
**HARDWARE SETUP / PLATFORM:** CubeMX + Keil MDK + STM32 Development Board + WemosD1 WiFi Module  
**TASK NO:** 11A 

---

### Circuit Diagram
![Circuit Diagram](../CAT11.png)

---

### Core Logic

* **1. Polling for Incoming Network Requests**
  Listens over UART to detect the `+IPD` tag, which indicates incoming data from a client browser, and extracts the connection ID required to reply to that specific client.
  ```c
  char *ipd_ptr = strstr(rx_data, "+IPD,"); 
  if(ipd_ptr != NULL) {
      int link_id = 0;
      sscanf(ipd_ptr, "+IPD,%d,", &link_id);
  ```

* **2. Constructing the Static HTTP Response**
  Builds a strictly formatted HTTP/1.1 response string consisting of the status line, headers (dynamically calculating `Content-Length`), and the static HTML body requested by the lab manual.
  ```c
  char html_body[] = "<html>\n<body>\n<h1>STM32 IoT Node</h1>\n<p>Hello from STM32</p>\n</body>\n</html>";
  int total_len = snprintf(http_response, sizeof(http_response),
      "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\nContent-Length: %d\r\n\r\n%s", 
      strlen(html_body), html_body);
  ```

* **3. CIPSEND and Transmission Synchronization**
  Sends the `AT+CIPSEND` command with the appropriate length, waits for the `>` prompt from the ESP8266 to ensure it is ready to receive data, and transmits the HTTP payload before closing the connection.
  ```c
  snprintf(send_cmd, sizeof(send_cmd), "AT+CIPSEND=%d,%d\r\n", link_id, total_len);
  HAL_UART_Transmit(&huart1, (uint8_t*)send_cmd, strlen(send_cmd), HAL_MAX_DELAY);
  
  // Wait for '>' prompt before sending payload...
  HAL_UART_Transmit(&huart1, (uint8_t*)http_response, total_len, HAL_MAX_DELAY);
  ```

---

### Inference
The STM32 successfully parsed an incoming TCP request from a standard web browser, extracted the routing link ID, and transmitted a correctly formatted HTTP/1.1 response containing static HTML. The two-step transmission process successfully synchronized with the ESP8266's data prompt (`>`).