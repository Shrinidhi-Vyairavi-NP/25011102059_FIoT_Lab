# EXP: Continuous Lab Evaluation-14

**DATE:** 27/10/2026

**NAME OF EXP:** Sensor Interfacing (BME280) – (MQTT) – Node Analytics: BME280 Sensor Interfacing Using $I^2C$ and (MQTT) using Custom Built ESP8266 API

## HARDWARE SETUP / PLATFORM

* **Tools & Hardware:** STM32CubeMX, Keil MDK, STM32 BluePill (STM32F103C8T6), Debugger, BME280 Sensor ($I^2C$), ESP8266 Wi-Fi Module (UART)
* **Libraries:** Custom Built ESP8266 API (MQTT), Bosch BME280 API, STM32 HAL Libraries, Node-RED

## TASK 13

## Circuit Diagram

![Circuit Diagram](CAT14.png)

## Core Logic

* **BME280 $I^2C$ Initialization:** Bind HAL $I^2C$ read/write functions to the Bosch `bme280_dev` structure (`0x76 << 1`) and set normal power mode with oversampling enabled.
* **Wi-Fi & MQTT Handshake:** Initialize `ESPLib` in Station Mode, connect to Wi-Fi and `broker.emqx.io` (port `1883`), and establish a clean MQTT session.
* **MQTT Timer Hook:** Call `MQTTTimer()` inside `SysTick_Handler()` (`stm32f1xx_it.c`) to keep internal MQTT connection timers active.
* **JSON Packaging & Publishing:** Periodically poll compensated sensor data, format temperature, humidity, and pressure ($\text{Pa}/100$) into a JSON payload, and publish via `MQTT_publishTopic()` for Node-RED visualization.

```c
// 1. MQTT Session Setup (in main.c)
MQTT_InitTypeDef_t mqttInitConfig = {
    .clientID    = "STM32_BME280_Node_SNUC_01",
    .keepAlive   = 500,
    .connectFlag = CleanSession
};
MQTT_connectBroker(&mqttInitConfig);

// 2. Sensor Polling & JSON Telemetry Publish (inside while(1))
bme280_get_sensor_data(BME280_ALL, &comp_data, &bme280);
snprintf(payload, sizeof(payload), 
         "{\"status\":\"success\",\"temperature\":%.2f,\"humidity\":%.2f,\"pressure\":%.2f}", 
         comp_data.temperature, comp_data.humidity, comp_data.pressure / 100.0);
MQTT_publishTopic("stm32/environment", payload, 0);

// 3. MQTT Keep-Alive Timer Hook (in stm32f1xx_it.c)
void SysTick_Handler(void) {
    MQTTTimer();
    HAL_IncTick();
}
```

## INFERENCE

The STM32 successfully read compensated environmental data from the BME280 sensor over $I^2C$, formatted the readings into JSON strings, and published them via the custom ESP8266 MQTT API to `broker.emqx.io`, allowing Node-RED to parse and visualize live sensor metrics accurately.