# Continuous Lab Evaluation-3

**EXP:** Continuous Lab Evaluation-3  
**DATE:** 27/07/2026  
**NAME OF EXP:** Timer-Based LED Pattern Generator and Button Interrupt Controller  
**HARDWARE SETUP / PLATFORM:** 8051 Tool Chain setup Development Board - AT89S52 + Keil C51 + Four LEDs + Push-button + Multimeter  
**TASK NO:** 3  

---

### Circuit Diagram
![Circuit Diagram](../CAT3.png)

---

### Core Logic

* **1. Active-Low Port Masking & LED Output Control**
  Inverts the target 4-bit pattern for active-low LED driving while masking the upper nibble (`0xF0`) so the push-button input pin remains pulled high and undisturbed.
  ```c
  void set_leds(unsigned char pattern_nibble) {
      LED_PORT = (LED_PORT & 0xF0) | (~pattern_nibble & 0x0F);
  }
  ```

* **2. Switch Debouncing & Single-Edge Detection**
  Filters mechanical switch bounce using a 20 ms confirmation delay and holds execution until button release (`while (BTN == 0)`), guaranteeing exactly one state transition per physical press.
  ```c
  unsigned char check_button_pressed(void) {
      if (BTN == 0) {
          delay_ms(20);
          if (BTN == 0) {
              while (BTN == 0);
              delay_ms(20);
              return 1;
          }
      }
      return 0;
  }
  ```

* **3. Four-State Finite State Machine (FSM)**
  Cycles through the four required LED operating modes on each valid button press, wrapping back from Mode 4 to Mode 1 and resetting pattern indices on every transition.
  ```c
  switch (mode) {
      case 0: set_leds(0x00); break;                             // Initial: All OFF
      case 1: set_leds(1 << step); step = (step + 1) % 4; break; // Running LED
      case 2: set_leds(count & 0x0F); count = (count + 1) % 16; break; // Binary Counter
      case 3: set_leds(step ? 0x05 : 0x0A); step ^= 1; break;    // Alternating (1010/0101)
      case 4: set_leds(step ? 0x00 : 0x0F); step ^= 1; break;    // All-LED Flash
  }
  ```

---

### Inference
A bare-metal four-state LED pattern controller was implemented on the AT89S52 microcontroller using direct memory-mapped SFR access in Keil C51 without external APIs or libraries. Combining 20 ms software debouncing with release-edge detection eliminated spurious triggers, ensuring each physical push-button press produced exactly one deterministic transition across the running LED, binary counter, alternating, and flashing modes.