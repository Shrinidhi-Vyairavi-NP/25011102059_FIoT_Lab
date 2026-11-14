# Continuous Lab Evaluation-5 

**EXP:** Continuous Lab Evaluation-5  
**DATE:** 10/08/2026  
**NAME OF EXP:** Hardware Timer and Interrupt-Based Timing for LED Fading, Pulse Generation, and Stepper Motor Control  
**HARDWARE SETUP / PLATFORM:** Keil uVision + 8051 Development Board (AT89S52, 11.0592 MHz) + LED + Oscilloscope + Multimeter  
**TASK NO:** 2 - LED Fade 

---

### Circuit Diagram
![Circuit Diagram](../CAT5B.png)

---

### Core Logic

* **1. Timer 0 High-Frequency Time Base Initialization**
  Configures Timer 0 in 16-bit Mode 1 (`TMOD = 0x01`) with preload `0xFFF7` and enables Timer 0 and global interrupts (`ET0`, `EA`) so PWM timing runs independently of `while(1)`.
  ```c
  TMOD = 0x01;
  TH0 = 0xFF;
  TL0 = 0xF7;

  ET0 = 1;
  EA  = 1;
  TR0 = 1;
  ```

* **2. Non-Blocking Software PWM Generation**
  Inside `timer0_ISR`, reloads Timer 0 and compares a 100-step tick counter (`pwm_count`) against `duty` to switch the LED ON or OFF fast enough to eliminate visible flicker.
  ```c
  void timer0_ISR(void) interrupt 1
  {
      TH0 = 0xFF;
      TL0 = 0xF7;

      LED = (pwm_count < duty) ? 1 : 0;
      pwm_count++;
  ```

* **3. Gradual Duty-Cycle Fading ($0\% \rightarrow 100\% \rightarrow 0\%$)**
  Every 10 complete PWM cycles (`fade_count >= 10`), updates `duty` in $10\%$ increments and reverses `direction` at the $100\%$ and $0\%$ limits for smooth LED fading.
  ```c
      if (pwm_count >= 100) {
          pwm_count = 0;
          if (++fade_count >= 10) {
              fade_count = 0;
              if (direction == 1) {
                  duty += 10;
                  if (duty >= 100) direction = 0;
              } else {
                  duty -= 10;
                  if (duty == 0) direction = 1;
              }
          }
      }
  }
  ```

---

### Inference
Modulating the software PWM duty cycle between $0\%$ and $100\%$ via Timer 0 interrupts produced continuous, flicker-free LED brightness fading without blocking CPU execution.